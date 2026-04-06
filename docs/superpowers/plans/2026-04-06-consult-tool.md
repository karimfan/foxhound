# Consult Tool Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `consult`, a CLI tool that identifies code experts via git history and lets LLM agents ask them questions via Slack.

**Architecture:** Multi-file Go tool in `consult/` directory. `main.go` handles CLI dispatch; `git.go` does blame/log analysis and expert ranking; `slack.go` wraps the Slack API (HTTP, no SDK); `message.go` formats consultation messages; `session.go` manages async session state in `.consult/`; skill template teaches LLMs when to consult humans.

**Tech Stack:** Go 1.25, stdlib only (HTTP for Slack API), git CLI

---

### Task 1: Project Structure and CLI Skeleton

**Files:**
- Create: `consult/main.go`

- [ ] **Step 1: Create directory and main.go with CLI parsing**

```go
package main

import (
	"fmt"
	"os"
	"strings"
)

func main() {
	if len(os.Args) < 2 {
		printUsage()
		os.Exit(1)
	}

	cmd := os.Args[1]
	args := os.Args[2:]

	switch cmd {
	case "who":
		cmdWho(args)
	case "ask":
		cmdAsk(args)
	case "propose":
		cmdPropose(args)
	case "check":
		cmdCheck(args)
	case "sessions":
		cmdSessions(args)
	case "--update-skills":
		cmdUpdateSkills(args)
	case "--help", "-h":
		printUsage()
	default:
		fmt.Fprintf(os.Stderr, "unknown command: %s\n", cmd)
		printUsage()
		os.Exit(1)
	}
}

type cmdFlags struct {
	file     string
	dir      string
	question string
	diff     string
	session  string
	dryRun   bool
	repoRoot string
}

func parseFlags(args []string) cmdFlags {
	var f cmdFlags
	for i := 0; i < len(args); i++ {
		a := args[i]
		switch {
		case a == "--dry-run":
			f.dryRun = true
		case a == "--file" && i+1 < len(args):
			i++
			f.file = args[i]
		case a == "--dir" && i+1 < len(args):
			i++
			f.dir = args[i]
		case a == "--question" && i+1 < len(args):
			i++
			f.question = args[i]
		case a == "--diff" && i+1 < len(args):
			i++
			f.diff = args[i]
		case a == "--session" && i+1 < len(args):
			i++
			f.session = args[i]
		case a == "--repo" && i+1 < len(args):
			i++
			f.repoRoot = args[i]
		case strings.HasPrefix(a, "-"):
			fmt.Fprintf(os.Stderr, "unknown flag: %s\n", a)
			os.Exit(1)
		default:
			// Positional: treat as file path if no --file given.
			if f.file == "" && f.dir == "" {
				f.file = a
			}
		}
	}
	return f
}

func resolveRepoRoot(f cmdFlags) string {
	if f.repoRoot != "" {
		return f.repoRoot
	}
	// Default to current directory.
	wd, _ := os.Getwd()
	return wd
}

func targetPath(f cmdFlags) string {
	if f.file != "" {
		return f.file
	}
	return f.dir
}

// Stub subcommands — implemented in later tasks.

func cmdWho(args []string) {
	f := parseFlags(args)
	target := targetPath(f)
	if target == "" {
		fmt.Fprintln(os.Stderr, "error: --file or --dir required")
		os.Exit(1)
	}
	repoRoot := resolveRepoRoot(f)
	experts, err := analyzeExperts(repoRoot, target)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
	printExperts(experts)
}

func cmdAsk(args []string) {
	fmt.Fprintln(os.Stderr, "ask: not yet implemented")
	os.Exit(1)
}

func cmdPropose(args []string) {
	fmt.Fprintln(os.Stderr, "propose: not yet implemented")
	os.Exit(1)
}

func cmdCheck(args []string) {
	fmt.Fprintln(os.Stderr, "check: not yet implemented")
	os.Exit(1)
}

func cmdSessions(args []string) {
	fmt.Fprintln(os.Stderr, "sessions: not yet implemented")
	os.Exit(1)
}

func cmdUpdateSkills(args []string) {
	fmt.Fprintln(os.Stderr, "update-skills: not yet implemented")
	os.Exit(1)
}

func printUsage() {
	fmt.Fprintln(os.Stderr, `Usage: consult <command> [flags]

Commands:
  who         Identify experts for a file or directory (no Slack needed)
  ask         Ask a question to the top expert via Slack
  propose     Propose a change for review via Slack (includes diff)
  check       Check for a response to a previous consultation
  sessions    List all consultation sessions

Flags:
  --file <path>      Target file to analyze
  --dir <path>       Target directory to analyze
  --question <text>  Question or context for the expert
  --diff <text>      Diff content to include (for propose)
  --session <id>     Session ID (for check)
  --repo <path>      Repository root (default: current directory)
  --dry-run          Show what would be sent without sending
  --update-skills    Update CLAUDE.md and AGENTS.md with consult skill

Examples:
  consult who --file internal/auth/handler.go
  consult ask --file internal/auth/handler.go --question "Is there a rate limit middleware?"
  consult propose --file internal/auth/handler.go --diff "$(git diff)" --question "Safe?"
  consult check --session abc123
  consult sessions`)
}
```

- [ ] **Step 2: Verify compilation**

Run: `go build ./consult/`
Expected: Will fail because `analyzeExperts` and `printExperts` are not defined yet. That's OK — we'll add stubs.

Add temporary stubs at the bottom of main.go to make it compile:

```go
// Temporary stubs — replaced by git.go in Task 2.
type expert struct{}

func analyzeExperts(repoRoot, target string) ([]expert, error) {
	return nil, fmt.Errorf("not implemented")
}

func printExperts(experts []expert) {}
```

Run: `go build ./consult/`
Expected: Compiles.

- [ ] **Step 3: Commit**

```bash
git add consult/main.go
git commit -m "feat(consult): add CLI skeleton with subcommand dispatch"
```

---

### Task 2: Git Analysis and Expert Ranking

**Files:**
- Create: `consult/git.go`
- Modify: `consult/main.go` (remove stubs)

- [ ] **Step 1: Create git.go with expert type and analysis**

```go
package main

import (
	"fmt"
	"os"
	"os/exec"
	"path/filepath"
	"sort"
	"strings"
)

type expert struct {
	Name       string
	Email      string
	SlackID    string
	Commits90d int
	Commits1y  int
	BlameLines int
	Score      float64
	LastCommit string
}

// analyzeExperts finds the top experts for a file or directory using git history.
func analyzeExperts(repoRoot, target string) ([]expert, error) {
	// Verify git repo.
	if _, err := os.Stat(filepath.Join(repoRoot, ".git")); err != nil {
		return nil, fmt.Errorf("not a git repository: %s", repoRoot)
	}

	// Determine if target is a file or directory.
	absTarget := filepath.Join(repoRoot, target)
	info, err := os.Stat(absTarget)
	if err != nil {
		return nil, fmt.Errorf("target not found: %s", target)
	}

	var files []string
	if info.IsDir() {
		files = listSourceFiles(repoRoot, target, 20)
	} else {
		files = []string{target}
	}

	if len(files) == 0 {
		return nil, fmt.Errorf("no source files found in %s", target)
	}

	// Count commits per author (last year).
	commits1y := countCommits(repoRoot, target, "1y")
	commits90d := countCommits(repoRoot, target, "90d")

	// Count blame lines per author (for files only, not dirs).
	blameLines := make(map[string]int)
	for _, f := range files {
		for email, lines := range blameFile(repoRoot, f) {
			blameLines[email] += lines
		}
	}

	// Merge all known authors.
	allAuthors := make(map[string]bool)
	for email := range commits1y {
		allAuthors[email] = true
	}
	for email := range commits90d {
		allAuthors[email] = true
	}
	for email := range blameLines {
		allAuthors[email] = true
	}

	// Score and rank.
	var experts []expert
	for email := range allAuthors {
		e := expert{
			Email:      email,
			Commits90d: commits90d[email],
			Commits1y:  commits1y[email],
			BlameLines: blameLines[email],
		}
		e.Score = float64(e.Commits90d)*3 + float64(e.Commits1y)*1 + float64(e.BlameLines)*0.5
		e.Name = getAuthorName(repoRoot, email)
		e.LastCommit = getLastCommitDate(repoRoot, email, target)
		experts = append(experts, e)
	}

	sort.Slice(experts, func(i, j int) bool {
		return experts[i].Score > experts[j].Score
	})

	// Return top 3.
	if len(experts) > 3 {
		experts = experts[:3]
	}

	return experts, nil
}

// countCommits counts commits per author email for a path within a time window.
func countCommits(repoRoot, path, since string) map[string]int {
	cmd := exec.Command("git", "-C", repoRoot, "log", "--format=%ae", "--since="+since, "--", path)
	out, err := cmd.Output()
	if err != nil {
		return nil
	}
	counts := make(map[string]int)
	for _, line := range strings.Split(strings.TrimSpace(string(out)), "\n") {
		line = strings.TrimSpace(line)
		if line != "" {
			counts[line]++
		}
	}
	return counts
}

// blameFile returns a map of author email → line count for a file.
func blameFile(repoRoot, filePath string) map[string]int {
	cmd := exec.Command("git", "-C", repoRoot, "blame", "--porcelain", "--", filePath)
	out, err := cmd.Output()
	if err != nil {
		return nil
	}
	counts := make(map[string]int)
	lines := strings.Split(string(out), "\n")
	for _, line := range lines {
		if strings.HasPrefix(line, "author-mail ") {
			email := strings.TrimPrefix(line, "author-mail ")
			email = strings.Trim(email, "<>")
			if email != "" && email != "not.committed.yet" {
				counts[email]++
			}
		}
	}
	return counts
}

// getAuthorName looks up the display name for a git author email.
func getAuthorName(repoRoot, email string) string {
	cmd := exec.Command("git", "-C", repoRoot, "log", "--format=%an", "--author="+email, "-1")
	out, err := cmd.Output()
	if err != nil {
		return email
	}
	name := strings.TrimSpace(string(out))
	if name == "" {
		return email
	}
	return name
}

// getLastCommitDate returns the date of the most recent commit by an author to a path.
func getLastCommitDate(repoRoot, email, path string) string {
	cmd := exec.Command("git", "-C", repoRoot, "log", "--format=%as", "--author="+email, "-1", "--", path)
	out, err := cmd.Output()
	if err != nil {
		return ""
	}
	return strings.TrimSpace(string(out))
}

// listSourceFiles returns up to maxFiles recently-modified source files in a directory.
func listSourceFiles(repoRoot, dir string, maxFiles int) []string {
	cmd := exec.Command("git", "-C", repoRoot, "log", "--format=", "--name-only", "--since=1y", "-100", "--", dir)
	out, err := cmd.Output()
	if err != nil {
		return nil
	}
	seen := make(map[string]bool)
	var files []string
	for _, line := range strings.Split(string(out), "\n") {
		line = strings.TrimSpace(line)
		if line == "" || seen[line] {
			continue
		}
		// Check file exists.
		if _, err := os.Stat(filepath.Join(repoRoot, line)); err == nil {
			seen[line] = true
			files = append(files, line)
			if len(files) >= maxFiles {
				break
			}
		}
	}
	return files
}

// printExperts prints a table of experts to stdout.
func printExperts(experts []expert) {
	if len(experts) == 0 {
		fmt.Println("No experts found for this path.")
		return
	}
	fmt.Printf("%-25s %-30s %8s %8s %8s %8s %12s\n", "Name", "Email", "90d", "1y", "Blame", "Score", "Last Commit")
	fmt.Println(strings.Repeat("-", 115))
	for _, e := range experts {
		fmt.Printf("%-25s %-30s %8d %8d %8d %8.1f %12s\n",
			truncate(e.Name, 25), truncate(e.Email, 30),
			e.Commits90d, e.Commits1y, e.BlameLines, e.Score, e.LastCommit)
	}
}

func truncate(s string, maxLen int) string {
	if len(s) <= maxLen {
		return s
	}
	return s[:maxLen-1] + "…"
}
```

- [ ] **Step 2: Remove stubs from main.go**

Remove the temporary stub types and functions (expert struct, analyzeExperts, printExperts) from the bottom of main.go.

- [ ] **Step 3: Verify compilation and test**

Run: `go build ./consult/ && ./consult who --file wikigen.go`
Expected: Compiles and prints expert table for wikigen.go.

- [ ] **Step 4: Commit**

```bash
git add consult/git.go consult/main.go
git commit -m "feat(consult): add git blame/log analysis and expert ranking"
```

---

### Task 3: Slack API Client

**Files:**
- Create: `consult/slack.go`

- [ ] **Step 1: Create slack.go with all Slack API functions**

```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"strings"
)

// lookupSlackUser resolves a git email to a Slack user ID and display name.
func lookupSlackUser(token, email string) (userID, displayName string, err error) {
	resp, err := slackGet(token, "https://slack.com/api/users.lookupByEmail", url.Values{"email": {email}})
	if err != nil {
		return "", "", err
	}
	var result struct {
		OK   bool `json:"ok"`
		User struct {
			ID      string `json:"id"`
			Profile struct {
				DisplayName string `json:"display_name"`
				RealName    string `json:"real_name"`
			} `json:"profile"`
		} `json:"user"`
		Error string `json:"error"`
	}
	if err := json.Unmarshal(resp, &result); err != nil {
		return "", "", fmt.Errorf("parse Slack response: %w", err)
	}
	if !result.OK {
		return "", "", fmt.Errorf("Slack lookup failed for %s: %s", email, result.Error)
	}
	name := result.User.Profile.DisplayName
	if name == "" {
		name = result.User.Profile.RealName
	}
	return result.User.ID, name, nil
}

// openDM opens a direct message channel with a Slack user.
func openDM(token, userID string) (channelID string, err error) {
	body := fmt.Sprintf(`{"users":"%s"}`, userID)
	resp, err := slackPost(token, "https://slack.com/api/conversations.open", body)
	if err != nil {
		return "", err
	}
	var result struct {
		OK      bool `json:"ok"`
		Channel struct {
			ID string `json:"id"`
		} `json:"channel"`
		Error string `json:"error"`
	}
	if err := json.Unmarshal(resp, &result); err != nil {
		return "", fmt.Errorf("parse Slack response: %w", err)
	}
	if !result.OK {
		return "", fmt.Errorf("Slack open DM failed: %s", result.Error)
	}
	return result.Channel.ID, nil
}

// postMessage sends a message to a Slack channel and returns the thread timestamp.
func postMessage(token, channel, text string) (threadTS string, err error) {
	payload := map[string]string{
		"channel": channel,
		"text":    text,
	}
	body, _ := json.Marshal(payload)
	resp, err := slackPost(token, "https://slack.com/api/chat.postMessage", string(body))
	if err != nil {
		return "", err
	}
	var result struct {
		OK    bool   `json:"ok"`
		TS    string `json:"ts"`
		Error string `json:"error"`
	}
	if err := json.Unmarshal(resp, &result); err != nil {
		return "", fmt.Errorf("parse Slack response: %w", err)
	}
	if !result.OK {
		return "", fmt.Errorf("Slack post failed: %s", result.Error)
	}
	return result.TS, nil
}

// getThreadReplies fetches replies to a Slack thread, excluding the original message.
func getThreadReplies(token, channel, threadTS string) ([]slackReply, error) {
	params := url.Values{
		"channel": {channel},
		"ts":      {threadTS},
	}
	resp, err := slackGet(token, "https://slack.com/api/conversations.replies", params)
	if err != nil {
		return nil, err
	}
	var result struct {
		OK       bool `json:"ok"`
		Messages []struct {
			User string `json:"user"`
			Text string `json:"text"`
			TS   string `json:"ts"`
		} `json:"messages"`
		Error string `json:"error"`
	}
	if err := json.Unmarshal(resp, &result); err != nil {
		return nil, fmt.Errorf("parse Slack response: %w", err)
	}
	if !result.OK {
		return nil, fmt.Errorf("Slack replies failed: %s", result.Error)
	}
	// Skip the first message (the original post).
	var replies []slackReply
	for i, msg := range result.Messages {
		if i == 0 {
			continue
		}
		replies = append(replies, slackReply{
			UserID: msg.User,
			Text:   msg.Text,
			TS:     msg.TS,
		})
	}
	return replies, nil
}

type slackReply struct {
	UserID string
	Text   string
	TS     string
}

// resolveExpertSlackIDs looks up Slack IDs for experts. Falls back to config if lookup fails.
func resolveExpertSlackIDs(token, repoRoot string, experts []expert) []expert {
	cfg := loadConsultConfig(repoRoot)
	for i := range experts {
		// Try config override first.
		if cfg != nil {
			if slackID, ok := cfg.UserMap[experts[i].Email]; ok {
				experts[i].SlackID = slackID
				continue
			}
		}
		// Try Slack email lookup.
		if token != "" {
			userID, name, err := lookupSlackUser(token, experts[i].Email)
			if err == nil {
				experts[i].SlackID = userID
				if experts[i].Name == "" || experts[i].Name == experts[i].Email {
					experts[i].Name = name
				}
			}
		}
	}
	return experts
}

// consultConfig represents the optional .consult.json configuration.
type consultConfig struct {
	UserMap        map[string]string `json:"user_map"`
	DefaultChannel string            `json:"default_channel"`
}

// loadConsultConfig loads .consult.json from the repo root, if it exists.
func loadConsultConfig(repoRoot string) *consultConfig {
	data, err := os.ReadFile(fmt.Sprintf("%s/.consult.json", repoRoot))
	if err != nil {
		return nil
	}
	var cfg consultConfig
	if err := json.Unmarshal(data, &cfg); err != nil {
		return nil
	}
	return &cfg
}

// slackGet performs a GET request to the Slack API.
func slackGet(token, apiURL string, params url.Values) ([]byte, error) {
	u := apiURL + "?" + params.Encode()
	req, err := http.NewRequest("GET", u, nil)
	if err != nil {
		return nil, err
	}
	req.Header.Set("Authorization", "Bearer "+token)
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return nil, fmt.Errorf("Slack API request failed: %w", err)
	}
	defer resp.Body.Close()
	return io.ReadAll(resp.Body)
}

// slackPost performs a POST request to the Slack API.
func slackPost(token, apiURL, body string) ([]byte, error) {
	req, err := http.NewRequest("POST", apiURL, strings.NewReader(body))
	if err != nil {
		return nil, err
	}
	req.Header.Set("Authorization", "Bearer "+token)
	req.Header.Set("Content-Type", "application/json; charset=utf-8")
	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return nil, fmt.Errorf("Slack API request failed: %w", err)
	}
	defer resp.Body.Close()
	return io.ReadAll(resp.Body)
}

// requireSlackToken returns the token or exits with an error.
func requireSlackToken() string {
	token := os.Getenv("SLACK_BOT_TOKEN")
	if token == "" {
		fmt.Fprintln(os.Stderr, "error: SLACK_BOT_TOKEN environment variable is not set")
		fmt.Fprintln(os.Stderr, "Required Slack bot scopes: chat:write, users:read.email, im:write")
		os.Exit(1)
	}
	return token
}
```

- [ ] **Step 2: Verify compilation**

Run: `go build ./consult/`
Expected: Compiles.

- [ ] **Step 3: Commit**

```bash
git add consult/slack.go
git commit -m "feat(consult): add Slack API client with identity resolution"
```

---

### Task 4: Message Builder

**Files:**
- Create: `consult/message.go`

- [ ] **Step 1: Create message.go with message formatting and wiki integration**

```go
package main

import (
	"fmt"
	"os"
	"path/filepath"
	"strings"
)

// buildAskMessage formats a consultation message for an "ask" request.
func buildAskMessage(question string, experts []expert, filePath, wikiExcerpt string) string {
	var sb strings.Builder
	sb.WriteString("🔍 *Code Consultation Request*\n\n")
	sb.WriteString(fmt.Sprintf("*File:* `%s`\n", filePath))
	sb.WriteString("*Type:* Question\n")
	sb.WriteString("*From:* LLM Agent (via consult)\n\n")
	sb.WriteString(fmt.Sprintf("*Question:*\n%s\n\n", question))

	if len(experts) > 0 {
		e := experts[0]
		sb.WriteString(fmt.Sprintf("*Why you:* You've made %d commits to this file in the last 90 days (%d in the last year).\n\n",
			e.Commits90d, e.Commits1y))
	}

	if wikiExcerpt != "" {
		sb.WriteString("*File summary (from wiki):*\n")
		sb.WriteString(fmt.Sprintf("> %s\n\n", wikiExcerpt))
	}

	sb.WriteString("━━━\n_Reply in this thread. The agent will poll for your response._")
	return sb.String()
}

// buildProposeMessage formats a consultation message for a "propose" request.
func buildProposeMessage(question, diff string, experts []expert, filePath, wikiExcerpt string) string {
	var sb strings.Builder
	sb.WriteString("🔍 *Code Consultation Request*\n\n")
	sb.WriteString(fmt.Sprintf("*File:* `%s`\n", filePath))
	sb.WriteString("*Type:* Proposed Change\n")
	sb.WriteString("*From:* LLM Agent (via consult)\n\n")
	sb.WriteString(fmt.Sprintf("*Question:*\n%s\n\n", question))

	if diff != "" {
		diffLines := strings.Split(diff, "\n")
		if len(diffLines) > 20 {
			diffLines = diffLines[:20]
			diffLines = append(diffLines, fmt.Sprintf("... (%d more lines omitted)", len(strings.Split(diff, "\n"))-20))
		}
		sb.WriteString("*Proposed diff:*\n```\n")
		sb.WriteString(strings.Join(diffLines, "\n"))
		sb.WriteString("\n```\n\n")
	}

	if len(experts) > 0 {
		e := experts[0]
		sb.WriteString(fmt.Sprintf("*Why you:* You've made %d commits to this file in the last 90 days (%d in the last year).\n\n",
			e.Commits90d, e.Commits1y))
	}

	if wikiExcerpt != "" {
		sb.WriteString("*File summary (from wiki):*\n")
		sb.WriteString(fmt.Sprintf("> %s\n\n", wikiExcerpt))
	}

	sb.WriteString("━━━\n_Reply in this thread. The agent will poll for your response._")
	return sb.String()
}

// loadWikiExcerpt tries to load the Overview section from the nearest wiki SUMMARY.md.
func loadWikiExcerpt(repoRoot, filePath string) string {
	// Try common wiki locations.
	dir := filepath.Dir(filePath)
	for _, wikiBase := range []string{"", "docs"} {
		summaryPath := filepath.Join(repoRoot, wikiBase, dir, "SUMMARY.md")
		data, err := os.ReadFile(summaryPath)
		if err != nil {
			continue
		}
		// Extract Overview section.
		content := string(data)
		marker := "## Overview"
		idx := strings.Index(content, marker)
		if idx < 0 {
			continue
		}
		start := idx + len(marker)
		if nl := strings.Index(content[start:], "\n"); nl >= 0 {
			start += nl + 1
		}
		rest := content[start:]
		end := strings.Index(rest, "\n## ")
		if end >= 0 {
			rest = rest[:end]
		}
		excerpt := strings.TrimSpace(rest)
		// Take first 2 sentences max.
		sentences := strings.SplitN(excerpt, ". ", 3)
		if len(sentences) > 2 {
			excerpt = sentences[0] + ". " + sentences[1] + "."
		}
		return excerpt
	}
	return ""
}
```

- [ ] **Step 2: Verify compilation**

Run: `go build ./consult/`
Expected: Compiles.

- [ ] **Step 3: Commit**

```bash
git add consult/message.go
git commit -m "feat(consult): add message builder with wiki excerpt integration"
```

---

### Task 5: Session Management

**Files:**
- Create: `consult/session.go`

- [ ] **Step 1: Create session.go with session CRUD**

```go
package main

import (
	"crypto/rand"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"os"
	"path/filepath"
	"sort"
	"strings"
	"time"
)

type session struct {
	ID           string    `json:"id"`
	CreatedAt    string    `json:"created_at"`
	File         string    `json:"file"`
	Question     string    `json:"question"`
	Type         string    `json:"type"` // "ask" or "propose"
	Experts      []expert  `json:"experts"`
	SlackChannel string    `json:"slack_channel"`
	SlackThread  string    `json:"slack_thread_ts"`
	Status       string    `json:"status"` // "pending" or "complete"
	Response     string    `json:"response,omitempty"`
}

func sessionDir(repoRoot string) string {
	return filepath.Join(repoRoot, ".consult")
}

func generateSessionID() string {
	b := make([]byte, 4)
	rand.Read(b)
	return hex.EncodeToString(b)
}

func createSession(repoRoot, file, question, sessionType string, experts []expert, channel, threadTS string) (session, error) {
	dir := sessionDir(repoRoot)
	os.MkdirAll(dir, 0755)

	s := session{
		ID:           generateSessionID(),
		CreatedAt:    time.Now().UTC().Format(time.RFC3339),
		File:         file,
		Question:     question,
		Type:         sessionType,
		Experts:      experts,
		SlackChannel: channel,
		SlackThread:  threadTS,
		Status:       "pending",
	}

	data, err := json.MarshalIndent(s, "", "  ")
	if err != nil {
		return s, err
	}
	path := filepath.Join(dir, s.ID+".json")
	return s, os.WriteFile(path, data, 0644)
}

func loadSession(repoRoot, sessionID string) (session, error) {
	path := filepath.Join(sessionDir(repoRoot), sessionID+".json")
	data, err := os.ReadFile(path)
	if err != nil {
		return session{}, fmt.Errorf("session %s not found", sessionID)
	}
	var s session
	if err := json.Unmarshal(data, &s); err != nil {
		return session{}, fmt.Errorf("corrupt session file: %w", err)
	}
	return s, nil
}

func updateSession(repoRoot string, s session) error {
	data, err := json.MarshalIndent(s, "", "  ")
	if err != nil {
		return err
	}
	path := filepath.Join(sessionDir(repoRoot), s.ID+".json")
	return os.WriteFile(path, data, 0644)
}

func checkSessionResponse(token string, s session) (string, bool, error) {
	replies, err := getThreadReplies(token, s.SlackChannel, s.SlackThread)
	if err != nil {
		return "", false, err
	}
	if len(replies) == 0 {
		return "", false, nil
	}
	// Return the first reply.
	return replies[0].Text, true, nil
}

func listSessions(repoRoot string) ([]session, error) {
	dir := sessionDir(repoRoot)
	entries, err := os.ReadDir(dir)
	if err != nil {
		if os.IsNotExist(err) {
			return nil, nil
		}
		return nil, err
	}
	var sessions []session
	for _, e := range entries {
		if !strings.HasSuffix(e.Name(), ".json") {
			continue
		}
		data, err := os.ReadFile(filepath.Join(dir, e.Name()))
		if err != nil {
			continue
		}
		var s session
		if err := json.Unmarshal(data, &s); err != nil {
			continue
		}
		sessions = append(sessions, s)
	}
	sort.Slice(sessions, func(i, j int) bool {
		return sessions[i].CreatedAt > sessions[j].CreatedAt
	})
	return sessions, nil
}

func printSessions(sessions []session) {
	if len(sessions) == 0 {
		fmt.Println("No consultation sessions found.")
		return
	}
	fmt.Printf("%-10s %-40s %-10s %-10s %-20s\n", "ID", "File", "Type", "Status", "Created")
	fmt.Println(strings.Repeat("-", 95))
	for _, s := range sessions {
		fmt.Printf("%-10s %-40s %-10s %-10s %-20s\n",
			s.ID, truncate(s.File, 40), s.Type, s.Status, s.CreatedAt[:19])
	}
}
```

- [ ] **Step 2: Verify compilation**

Run: `go build ./consult/`
Expected: Compiles.

- [ ] **Step 3: Commit**

```bash
git add consult/session.go
git commit -m "feat(consult): add session management for async Slack conversations"
```

---

### Task 6: Wire All Subcommands

**Files:**
- Modify: `consult/main.go` — replace stub subcommands with real implementations

- [ ] **Step 1: Replace cmdAsk**

```go
func cmdAsk(args []string) {
	f := parseFlags(args)
	target := targetPath(f)
	if target == "" {
		fmt.Fprintln(os.Stderr, "error: --file or --dir required")
		os.Exit(1)
	}
	if f.question == "" {
		fmt.Fprintln(os.Stderr, "error: --question required")
		os.Exit(1)
	}

	repoRoot := resolveRepoRoot(f)
	experts, err := analyzeExperts(repoRoot, target)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
	if len(experts) == 0 {
		fmt.Fprintln(os.Stderr, "No experts found for this path.")
		os.Exit(1)
	}

	wikiExcerpt := loadWikiExcerpt(repoRoot, target)
	msg := buildAskMessage(f.question, experts, target, wikiExcerpt)

	if f.dryRun {
		fmt.Fprintln(os.Stderr, "--- DRY RUN: Message that would be sent ---")
		fmt.Fprintln(os.Stderr, msg)
		fmt.Fprintln(os.Stderr, "--- Experts ---")
		printExperts(experts)
		return
	}

	token := requireSlackToken()
	experts = resolveExpertSlackIDs(token, repoRoot, experts)

	topExpert := experts[0]
	if topExpert.SlackID == "" {
		fmt.Fprintf(os.Stderr, "error: could not resolve Slack user for %s (%s)\n", topExpert.Name, topExpert.Email)
		fmt.Fprintln(os.Stderr, "Add a mapping to .consult.json or verify the email matches their Slack profile.")
		os.Exit(1)
	}

	channelID, err := openDM(token, topExpert.SlackID)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error opening DM: %v\n", err)
		os.Exit(1)
	}

	threadTS, err := postMessage(token, channelID, msg)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error sending message: %v\n", err)
		os.Exit(1)
	}

	sess, err := createSession(repoRoot, target, f.question, "ask", experts, channelID, threadTS)
	if err != nil {
		fmt.Fprintf(os.Stderr, "warning: could not save session: %v\n", err)
	}

	fmt.Printf("Consultation sent to %s (%s)\n", topExpert.Name, topExpert.Email)
	fmt.Printf("Session ID: %s\n", sess.ID)
	fmt.Printf("Check for response: consult check --session %s\n", sess.ID)
}
```

- [ ] **Step 2: Replace cmdPropose**

```go
func cmdPropose(args []string) {
	f := parseFlags(args)
	target := targetPath(f)
	if target == "" {
		fmt.Fprintln(os.Stderr, "error: --file or --dir required")
		os.Exit(1)
	}
	if f.question == "" {
		fmt.Fprintln(os.Stderr, "error: --question required")
		os.Exit(1)
	}

	repoRoot := resolveRepoRoot(f)
	experts, err := analyzeExperts(repoRoot, target)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
	if len(experts) == 0 {
		fmt.Fprintln(os.Stderr, "No experts found for this path.")
		os.Exit(1)
	}

	wikiExcerpt := loadWikiExcerpt(repoRoot, target)
	msg := buildProposeMessage(f.question, f.diff, experts, target, wikiExcerpt)

	if f.dryRun {
		fmt.Fprintln(os.Stderr, "--- DRY RUN: Message that would be sent ---")
		fmt.Fprintln(os.Stderr, msg)
		fmt.Fprintln(os.Stderr, "--- Experts ---")
		printExperts(experts)
		return
	}

	token := requireSlackToken()
	experts = resolveExpertSlackIDs(token, repoRoot, experts)

	topExpert := experts[0]
	if topExpert.SlackID == "" {
		fmt.Fprintf(os.Stderr, "error: could not resolve Slack user for %s (%s)\n", topExpert.Name, topExpert.Email)
		fmt.Fprintln(os.Stderr, "Add a mapping to .consult.json or verify the email matches their Slack profile.")
		os.Exit(1)
	}

	channelID, err := openDM(token, topExpert.SlackID)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error opening DM: %v\n", err)
		os.Exit(1)
	}

	threadTS, err := postMessage(token, channelID, msg)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error sending message: %v\n", err)
		os.Exit(1)
	}

	sess, err := createSession(repoRoot, target, f.question, "propose", experts, channelID, threadTS)
	if err != nil {
		fmt.Fprintf(os.Stderr, "warning: could not save session: %v\n", err)
	}

	fmt.Printf("Proposal sent to %s (%s)\n", topExpert.Name, topExpert.Email)
	fmt.Printf("Session ID: %s\n", sess.ID)
	fmt.Printf("Check for response: consult check --session %s\n", sess.ID)
}
```

- [ ] **Step 3: Replace cmdCheck**

```go
func cmdCheck(args []string) {
	f := parseFlags(args)
	if f.session == "" {
		fmt.Fprintln(os.Stderr, "error: --session required")
		os.Exit(1)
	}

	repoRoot := resolveRepoRoot(f)
	sess, err := loadSession(repoRoot, f.session)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}

	if sess.Status == "complete" {
		fmt.Println("Response (already received):")
		fmt.Println(sess.Response)
		return
	}

	token := requireSlackToken()
	response, hasReply, err := checkSessionResponse(token, sess)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error checking response: %v\n", err)
		os.Exit(1)
	}

	if !hasReply {
		fmt.Println("No response yet.")
		return
	}

	sess.Status = "complete"
	sess.Response = response
	updateSession(repoRoot, sess)

	fmt.Println("Response received:")
	fmt.Println(response)
}
```

- [ ] **Step 4: Replace cmdSessions**

```go
func cmdSessions(args []string) {
	f := parseFlags(args)
	repoRoot := resolveRepoRoot(f)
	sessions, err := listSessions(repoRoot)
	if err != nil {
		fmt.Fprintf(os.Stderr, "error: %v\n", err)
		os.Exit(1)
	}
	printSessions(sessions)
}
```

- [ ] **Step 5: Verify compilation and test dry-run**

Run: `go build ./consult/ && ./consult ask --dry-run --file wikigen.go --question "test question"`
Expected: Compiles, shows dry-run message and experts.

- [ ] **Step 6: Commit**

```bash
git add consult/main.go
git commit -m "feat(consult): wire ask, propose, check, sessions subcommands"
```

---

### Task 7: Skill File Template

**Files:**
- Create: `consult/templates/CONSULT_SKILL.md`
- Modify: `consult/main.go` — add embed and writeSkillFiles

- [ ] **Step 1: Create the skill template**

Create `consult/templates/CONSULT_SKILL.md`:

```markdown
# Expert Consultation via consult

This project has the `consult` tool installed. It lets you ask human experts for help via Slack when you're unsure about code you're modifying.

## When to consult a human

Use this three-layer decision framework:

### Layer 1 — Wiki sufficient (don't consult)
The wiki documents the code well, the change is straightforward, and you understand what you're doing. Just use the wiki and proceed.

### Layer 2 — Source sufficient (don't consult)
You can understand the code by reading the source files. The change is within well-documented, well-tested code. Proceed.

### Layer 3 — Consult a human
You should consult when:
- You're modifying code you haven't seen before in a **high-churn area**
- You're changing a **public API** that other packages depend on
- Code has comments like "DON'T CHANGE", "HACK", "WORKAROUND", or "TODO: fragile"
- The wiki says the directory is **internal** but you need to import it externally
- You're **unsure if your approach is correct** and the cost of being wrong is high
- You need to understand **why** something was built a certain way (tribal knowledge)

### When NOT to consult
- Adding tests, fixing typos, formatting changes
- Well-documented code with clear patterns
- Changes within a directory marked "internal" that nothing depends on
- Straightforward feature additions following existing patterns

## How to use

### Step 1: Preview experts (no Slack needed)
```
consult who --file path/to/file.go
```
This shows who knows about the file without contacting anyone. Use this to assess whether consultation is worthwhile.

### Step 2: Ask or propose
```
# Ask a question
consult ask --file path/to/file.go --question "Is there a rate limit middleware I should use?"

# Propose a change with a diff
consult propose --file path/to/file.go --diff "$(git diff)" --question "Does this look safe?"
```

### Step 3: Check for response
```
consult check --session <session-id>
```
The expert replies in Slack. Poll periodically — humans may take minutes or hours.

### Step 4: Continue work
Incorporate the expert's feedback into your changes. If no response after a reasonable time, proceed with your best judgment and note the consultation attempt in the PR.

## Formulating good questions

**Good:** "I'm adding rate limiting to the login handler. I see there's a middleware pattern in pkg/middleware/ — should I use that, or is there a reason auth has its own inline checks?"

**Bad:** "How does this code work?"

Be specific. Include what you're trying to do, what you've already found, and what you're unsure about.
```

- [ ] **Step 2: Add embed and writeSkillFiles to main.go**

Add near the top of main.go (after the imports):

```go
import (
	"embed"
	// ... existing imports ...
)

//go:embed templates/CONSULT_SKILL.md
var skillTemplate embed.FS
```

Then implement `cmdUpdateSkills`:

```go
const (
	consultSectionStart = "<!-- consult:start -->"
	consultSectionEnd   = "<!-- consult:end -->"
)

func cmdUpdateSkills(args []string) {
	f := parseFlags(args)
	repoRoot := resolveRepoRoot(f)

	data, err := skillTemplate.ReadFile("templates/CONSULT_SKILL.md")
	if err != nil {
		fmt.Fprintf(os.Stderr, "error reading skill template: %v\n", err)
		os.Exit(1)
	}

	section := fmt.Sprintf("%s\n%s\n%s", consultSectionStart, strings.TrimSpace(string(data)), consultSectionEnd)

	for _, name := range []string{"CLAUDE.md", "AGENTS.md"} {
		outPath := filepath.Join(repoRoot, name)
		existing, err := os.ReadFile(outPath)
		if err != nil {
			// File doesn't exist — create with just our section.
			os.WriteFile(outPath, []byte(section+"\n"), 0644)
			fmt.Fprintf(os.Stderr, "Created %s\n", outPath)
			continue
		}

		content := string(existing)
		startIdx := strings.Index(content, consultSectionStart)
		endIdx := strings.Index(content, consultSectionEnd)

		if startIdx >= 0 && endIdx >= 0 && endIdx > startIdx {
			// Replace existing section.
			updated := content[:startIdx] + section + content[endIdx+len(consultSectionEnd):]
			os.WriteFile(outPath, []byte(updated), 0644)
			fmt.Fprintf(os.Stderr, "Updated consult section in %s\n", outPath)
		} else {
			// Append.
			separator := "\n\n"
			if strings.HasSuffix(content, "\n\n") {
				separator = ""
			} else if strings.HasSuffix(content, "\n") {
				separator = "\n"
			}
			os.WriteFile(outPath, []byte(content+separator+section+"\n"), 0644)
			fmt.Fprintf(os.Stderr, "Added consult section to %s\n", outPath)
		}
	}
	fmt.Fprintln(os.Stderr, "Skill files updated.")
}
```

- [ ] **Step 3: Verify compilation**

Run: `go build ./consult/`
Expected: Compiles.

- [ ] **Step 4: Commit**

```bash
git add consult/templates/CONSULT_SKILL.md consult/main.go
git commit -m "feat(consult): add skill template and --update-skills command"
```

---

### Task 8: Validation

**Files:** None (testing only)

- [ ] **Step 1: Build**

Run: `go build -o /tmp/consult ./consult/`
Expected: Binary built.

- [ ] **Step 2: Test help**

Run: `/tmp/consult --help`
Expected: Shows all subcommands and flags.

- [ ] **Step 3: Test who command**

Run: `/tmp/consult who --file wikigen.go`
Expected: Prints expert table with name, email, commits, blame lines, score.

- [ ] **Step 4: Test dry-run ask**

Run: `/tmp/consult ask --dry-run --file wikigen.go --question "Is this file getting too large?"`
Expected: Shows formatted Slack message and expert table, no SLACK_BOT_TOKEN needed.

- [ ] **Step 5: Test dry-run propose**

Run: `/tmp/consult propose --dry-run --file wikigen.go --diff "test diff content" --question "Safe?"`
Expected: Shows formatted message with diff included.

- [ ] **Step 6: Test sessions with no sessions**

Run: `/tmp/consult sessions`
Expected: "No consultation sessions found."

- [ ] **Step 7: Test missing token**

Run: `unset SLACK_BOT_TOKEN && /tmp/consult ask --file wikigen.go --question "test"`
Expected: Error message about SLACK_BOT_TOKEN with required scopes.

- [ ] **Step 8: Run go vet**

Run: `go vet ./consult/`
Expected: No issues.

- [ ] **Step 9: Commit any fixes**

```bash
git add consult/
git commit -m "fix(consult): address issues found during validation"
```
