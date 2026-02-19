# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a personal knowledge management repository managed with Obsidian. All content is in Japanese. The repository consolidates project documentation, technical notes, work logs, and web clippings.

## Folder Structure

```
docs/
├─ projects/    # プロジェクト別ドキュメント (公開)
└─ tech/        # 技術関連のナレッジ・トラブルシューティング (公開)

log/            # 作業メモ・個人用ログ (非公開・gitignore対象)
├─ all-tasks.md     # マスタータスクリスト
├─ dashborad.md     # ダッシュボード
└─ meeting/         # 会議メモ (日付別: YYYY-MM-DD.md)

template/       # テンプレートファイル (公開)
└─ daily-note.md    # デイリーノートテンプレート

Clippings/      # Web記事のクリッピング (非公開・gitignore対象)
```

## Key Architectural Points

### Privacy Zones

- **Public (committed)**: `docs/`, `template/`, `README.md`
- **Private (gitignored)**: `log/`, `Clippings/`, `.obsidian/`, `.env`

When creating or modifying files, respect this privacy boundary. Project documentation goes in `docs/projects/`, never in `log/`.

### Template System

`template/daily-note.md` is structured as a daily work log with predefined project sections and a Todo section. This template reflects the active projects tracked in this repository.

### Task Management

`log/all-tasks.md` serves as the master task list with three sections:
- **Qlife Inbox**: One-time tasks and backlog items
- **Regular**: Recurring meetings and daily tasks (uses Obsidian tasks plugin syntax with 🔁 recurring, 📅 dates, ✅ completion)
- **Tech**: Learning and book reading goals

Meeting notes are stored in `log/meeting/YYYY-MM-DD.md`.

### Naming Conventions

- Project folders use lowercase with hyphens or Japanese characters
- Documentation files use descriptive Japanese names
- Meeting logs follow ISO date format: `YYYY-MM-DD.md`

## Working with This Repository

### Adding Project Documentation

Place new project docs in `docs/projects/<project-name>/`. If the project doesn't have a folder yet, create one. Single-file projects can remain as standalone markdown files in `docs/projects/`.

### Adding Technical Notes

Technical troubleshooting, setup guides, and knowledge articles go in `docs/tech/<topic>/`. Organize by tool or technology (e.g., `docker/`, `datadog/`, `claude-code/`).

### Obsidian Integration

This repository is designed to be opened as an Obsidian vault. The `.obsidian/` folder contains Obsidian-specific configuration (workspace settings, plugins, themes) and is gitignored to avoid conflicts.

### Git Workflow

Standard git commands apply. The repository uses a single `main` branch. Since `log/` is gitignored, modifications to task lists and meeting notes won't appear in git status.
