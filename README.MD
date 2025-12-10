# Evergreen Tech Interview Notes

A public, evergreen repository for **refreshing core concepts**, **learning new topics**, and **preparing for interviews**.

This repo is organised by technology and each topic is written as a focused and structured Markdown lesson.

> Goal: **Fast refresh, deep understanding, and reusable notes** you can scan the night before an interview or when jumping back into a stack.

---

## 🧭 How to use this repo

- **Browse by technology**  
  Each folder (e.g. `dotnet-csharp`, `sql`) has its own `README` that acts as a mini learning path.

- **Pick your level**
  - `🏁` Beginner / fundamentals
  - `🚦` Intermediate / practical
  - `🔥` Advanced / deep dive

- **Use it three ways**
  1. **Interview prep** – focus on `99-interview-checklist.md` in each folder.
  2. **Refresh** – scan the “Fundamentals” and “Common pitfalls” sections in lessons.
  3. **Learn** – follow the lessons in order within a folder.

---

## 📁 Structure

| Area            | Folder          | Description                                      |
|-----------------|-----------------|--------------------------------------------------|
| .NET / C#       | `dotnet-csharp` | Language, runtime, patterns, interview drills    |
| SQL             | `sql`           | Querying, indexing, transactions, design         |
| React           | `react`         | Core concepts, hooks, patterns, performance      |
| Microsoft Azure | `azure`         | Core services, hosting, networking, security     |
| Kubernetes      | `kubernetes`    | Concepts, workloads, networking, deployment      |

---

## 🧱 Lesson format

Every lesson follows a simple, repeatable structure:

- **Why this matters (especially in interviews)**
- **Core ideas and mental models**
- **Code / config examples**
- **Common pitfalls & FAQ**
- **Quick self-check questions**
- **Further practice ideas**

See `sample-lesson.md` for a sample lesson.

### Lesson details

#### Why this matters

Provide depth and include necessary explanations.

---

## 🛠 Working with this repo

### 1. Naming conventions

- Use a **numeric prefix** for ordering:
  - `01-`, `02-`, … for core lessons
  - `99-` for summary / checklists
- Use **kebab-case** for file names, e.g.  
  `03-async-await-and-tpl.md`, `02-indexes-and-query-plans.md`.

### 2. Difficulty tags

Inside each lesson, include difficulty + tags at the top:

```md
**Level:** 🚦 Intermediate  
**Tags:** `interview`, `performance`, `debugging`

