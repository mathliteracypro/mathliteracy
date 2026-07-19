# 🤝 Contribution Guidelines

Thank you for your interest in contributing to MathLiteracy Pro! To maintain high code quality, zero-bug runtime behaviors, and professional documentation standards, please follow these guidelines.

---

## 🏛️ Code Style & Quality Rules

To keep the application highly performant and maintainable:

1. **TypeScript Strict Mode**: Avoid using `any` as a type declaration. Always declare clear interfaces for custom objects and parameters (e.g. within `src/types/index.ts`).
2. **Component Modularity**: Never pool massive amounts of code into a single massive component file. Extract reusable page widgets to `/src/components/` and keep core routes lightweight.
3. **Tailwind Visual Uniformity**: Use established styling variables for colors and layout density. Avoid defining custom CSS strings outside of Tailwind utility classes.
4. **Hook Dependency Management**: Always clean up event listeners, intervals, and subscriptions when components unmount to prevent severe memory leaks.

---

## 🗂️ Git Branching & Workflow

Always create dynamic, task-focused feature branches off the master `main` branch:

```text
main (stable)
  └── dev (staging updates)
       ├── feature/add-inflation-quiz
       └── bugfix/re-render-profile-stats
```

### 1. Create a Branch
```bash
git checkout -b feature/add-inflation-quiz
```

### 2. Verify Your Changes (MANDATORY)
Before committing any changes, run the linter and compiler suite locally to guarantee your build compiles without issue:

```bash
npm run lint
npm run build
```

---

## 📝 Commit Message Formats

We adhere strictly to the **Conventional Commits** specification:

- `feat:` for new capabilities or sections (e.g. `feat: add dynamic study graphs to progress dashboard`)
- `fix:` for fixing runtime errors (e.g. `fix: sanitize Firestore Timestamp objects during sitemap builds`)
- `docs:` for documentation updates (e.g. `docs: document Firestore security rules schema`)
- `style:` for visual styling and layout tweaks (e.g. `style: adjust dashboard cards grid padding`)

Example Commit:
```bash
git commit -m "feat: add secure server side proxy for gemini math tutor requests"
```

---

## 📬 Pull Request (PR) Requirements

When opening a Pull Request:
- Provide a summary describing the functional and visual outcomes.
- Confirm that the linter (`tsc --noEmit`) compiles with 100% success.
- Link relevant issues being resolved.
- Wait for reviews from team leads before merging into the main branch.
