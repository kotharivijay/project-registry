# Project Registry

A personal catalogue of projects across all my PCs. Each project is logged as a
Markdown record and shown on a small website.

- **Live site:** https://kotharivijay.github.io/project-registry/
- **Add a project from the web:** open the site, fill the form, and copy the
  generated `project.md` / VS Code prompt.

## Repo layout

```
index.html               The web tool (form + generator + registered-projects list)
project.html             Reader page that renders one project record
projects/
  registry.json          The list that drives the home-page cards
  <ProjectId>.md         One record per project
  index.html             Redirect so /projects/ doesn't 404
README.md                This file
```

---

## How to add a project (instructions for Claude Code on any PC)

If you are Claude Code running on one of my computers and I ask you to
"log this project to my registry", do exactly this:

**Prerequisite:** this PC's `git` / `gh` must be able to push to
`kotharivijay/project-registry` (same GitHub account, or added as a collaborator).

1. **Get the repo.** If it isn't already cloned here:
   ```
   git clone https://github.com/kotharivijay/project-registry.git
   ```
   Otherwise `cd` into it and `git pull`.

2. **Create the record** at `projects/<ProjectId>.md`, where `<ProjectId>` is the
   project name with no spaces (e.g. `KidGuard`). Use this template:

   ```markdown
   # <Project Name>

   | Field | Value |
   |---|---|
   | System | <this PC, e.g. Office Dell (Windows 11)> |
   | Version | <e.g. 1.0> |
   | Status | <e.g. In progress> |
   | Location | `<full folder path on this PC>` |
   | Last updated | <YYYY-MM-DD> |

   ## Description

   <one or two lines on what it does>

   ## Complete info

   <tech stack, how to build/run, key files, known issues, next steps>
   ```

3. **Register it** by adding one object to the array in `projects/registry.json`:
   ```json
   {
     "id": "<ProjectId>",
     "name": "<Project Name>",
     "system": "<this PC>",
     "version": "<e.g. 1.0>",
     "updated": "<YYYY-MM-DD>"
   }
   ```
   Keep the JSON valid (commas between objects, no trailing comma).

4. **Commit and push:**
   ```
   git add -A
   git commit -m "Log <Project Name> to registry"
   git push origin main
   ```

5. The website updates automatically in about a minute. The new project appears
   under "Registered projects" on the home page.

Pull accurate details from the actual project where possible (version from the
build file, real folder path, real component/file names) rather than guessing.
