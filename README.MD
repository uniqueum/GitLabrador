# 🐾 GitLabrador: Cross-Platform GitLab Notifier

## 💡 Why GitLabrador?

GitLab’s `/todos` endpoint is powerful — but it’s trapped in the browser. GitLabrador sets it free.

### 🚫 What’s Missing in Existing Tools
- **GitLab’s own To-Do List** is web-based only — no native desktop notifications.  
- **Super Productivity** is a heavy-duty task manager with GitLab integration, but it’s not focused on just `/todos`, and it’s not a lightweight tray app.  
- **OpenTodoList** is a general-purpose task manager — not GitLab-specific, and lacks native GitLab push/MR awareness.
- **Email notifications** are available in GitLab, but they often get ignored or buried in inbox noise. GitLabrador delivers focused, native alerts developers actually see.

### ✅ What GitLabrador Brings
- Native OS notifications for GitLab `/todos`
- Lightweight, background-friendly architecture
- Cross-platform support (Windows, Linux, macOS)
- Dev-centric TUI for configuration, no GUI bloat
- Optional collaborator watchlist for special alerts

---

## 🖱️ Tray Tooltip Behavior

When GitLabrador is running as a background tray app (or menubar/panel item), hovering over the icon reveals a concise status summary:

- **Status Dot**: 🟢 Running or 🔴 Misconfigured (based on PAT and GitLab URL validity)
- **Polling Interval**: Displays the current polling frequency (e.g., `Polling every: 60s`)
- **Last Notification**: Shows the most recent GitLab event (e.g., `"MR from @alice in ProjectX"`)
- **Collaborator Watchlist**: Lists tracked usernames (e.g., `Watching: @alice, @bob`)

This tooltip provides at-a-glance insight into GitLabrador’s health and activity without launching a GUI.

---

## 🏗️ Project Structure

### 1. 📦 Data Layer: Models (The What)

These C# classes mirror the structure of the JSON objects returned by GitLab’s `/api/v4/todos`.

#### `TodoItem.cs`
```
public class TodoItem
{
    public int Id { get; set; }
    public string ActionName { get; set; } // e.g., "mentioned", "assigned"
    public string TargetType { get; set; } // e.g., "Issue", "MergeRequest"
    public DateTime CreatedAt { get; set; }
    public string WebUrl { get; set; }
    public Project Project { get; set; }
    public Author Author { get; set; }
}
```

#### `Project.cs`
```
public class Project
{
    public string NameWithNamespace { get; set; }
}
```

#### `Author.cs`
```
public class Author
{
    public string Username { get; set; }
}
```

---

### 2. 🌐 Service Layer: GitLab API Communication

Handles authentication and polling of GitLab’s `/todos` endpoint.

#### `IGitLabService.cs`
```
public interface IGitLabService
{
    Task<List<TodoItem>> GetPendingTodosAsync();
    Task MarkTodoAsDoneAsync(int todoId);
}
```

#### `GitLabHttpClient.cs`
- Uses `HttpClient` and `System.Text.Json`
- Authenticates via Private Access Token (PAT)
- Polls `/api/v4/todos?state=pending`

---

### 3. 🔔 Service Layer: OS Notification Bridge

Abstracts platform-specific notification delivery.

#### `INotificationService.cs`
```
public interface INotificationService
{
    void ShowTodoNotification(TodoItem todo);
}
```

#### Platform Implementations
- `WindowsNotificationService.cs` → Uses `Microsoft.Toolkit.Uwp.Notifications`
- `LinuxNotificationService.cs` → Uses `notify-send` via `Process.Start`
- `MacNotificationService.cs` → Uses `osascript` with AppleScript

---

### 4. ⏱️ Application Layer: Polling Engine

#### `PollerEngine.cs`
- Timer-based polling loop
- Deduplicates by tracking notified To-Do IDs
- Triggers `INotificationService` for new items

---

### 5. 🧰 Configuration Layer

Stores user preferences in a local config file.

#### `config.json` Example
```
{
  "GitLabUrl": "https://gitlab.example.com",
  "PersonalAccessToken": "glpat-abc123",
  "PollingIntervalSeconds": 60,
  "Watchlist": [
    { "Username": "alice", "NotifyOn": ["Push", "MergeRequest"] }
  ]
}
```

---

### 6. 🖥️ TUI Layer: Spectre.Console Settings Shell

A cross-platform terminal UI for configuring GitLabrador.

#### Features
- Prompt for GitLab URL, PAT, polling interval
- Manage collaborator watchlist
- Save config to file
- Optional CLI flags for headless setup

#### Sample Flow
```
┌────────────────────────────┐
│   GitLabrador Settings     │
├────────────────────────────┤
│ GitLab URL: [___________]  │
│ PAT:        [***********]  │
│ Polling Interval: [ 60s ] │
├────────────────────────────┤
│ Collaborator Watchlist     │
│ - @alice [Push, MR]        │
│ [+] Add Collaborator       │
│ [x] Remove Collaborator    │
├────────────────────────────┤
│ [ Save & Exit ]            │
└────────────────────────────┘
```

---

## 🌍 Cross-Platform Strategy

| OS       | Notification Method             | Notes |
|----------|----------------------------------|-------|
| Windows  | `Microsoft.Toolkit.Uwp.Notifications` | Rich toast support |
| Linux    | `notify-send` via `Process.Start`     | Works in most DEs |
| macOS    | `osascript` with AppleScript          | Native notification |

Platform detection via `RuntimeInformation.OSDescription` ensures the correct notifier is injected at runtime.

---

## 🧪 Future Enhancements

- GitHub `/notifications` support
- Persistent state via LiteDB or SQLite
- System tray integration (optional via Avalonia)
- CLI flags for scripting and automation
- Notification templates and themes

---

## 🐾 GitLabrador Philosophy

- **Minimalist**: No GUI bloat, no distractions  
- **Respectful**: No spammy alerts — deduplicated and contextual  
- **Dev-first**: Built for developers, by developers  
- **Extensible**: Clean architecture for future integrations  

---

## 📜 License

GitLabrador will be released under the **GNU GPLv3**.

> Free puppies for all. No proprietary leashes.

---

## 🚀 Release Status

**Coming Soon**  
GitLabrador is currently in active development. A public release will follow once security, stability, and documentation milestones are met.
