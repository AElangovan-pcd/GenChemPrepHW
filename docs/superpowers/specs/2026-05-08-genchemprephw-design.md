# GenChemPrepHW Design

## Purpose

GenChemPrepHW is an advanced college-level general chemistry homework platform for instructors and students. The platform prioritizes accuracy, reliability, and traceability. The OER text at `https://chem-139-oer-text-2026.onrender.com/` and the GitHub fallback source at `https://github.com/AElangovan-pcd/CHEM-139-OER-Text-2026/tree/main/HTML_Files/learners` are the primary alignment sources. Trusted public OER chemistry references may supplement them when needed.

## Roles

The first version supports three roles:

- Admin: manages users, global configuration, and platform-level settings.
- Instructor: manages courses, question review, assignments, due dates, attempts, feedback, and grade exports.
- Student: joins courses by invite code, completes homework, reviews feedback, and practices by topic.

## Core Architecture

The core system is a reviewed question bank. Content moves through source ingestion, AI-assisted draft generation, automated validation, instructor review, and delivery. Questions have explicit states: `draft`, `validated`, and `approved`. Only `approved` questions can be used in graded homework. Ungraded practice may use approved questions and clearly labeled validated AI-assisted variants.

## Content And Accuracy Pipeline

AI-generated questions must be original variants, not copied source problems. Each question stores its aligned source section, learning objective, prompt, correct answer, worked solution, grading rule, difficulty, tags, and validation history.

Automated validation checks include answer recomputation when possible, unit compatibility, significant figures, tolerance rules, formula consistency, duplicate risk, source-topic alignment, and worked-solution consistency. Instructor approval is the final gate before graded use.

## Homework Workflow

Instructors create courses and invite students with course codes. Assignments are built from approved question-bank items and include open date, due date, allowed attempts, randomized order, late policy, point values, and feedback timing.

The first version supports multiple choice, true/false, matching, numeric answers with tolerance, units-aware numeric answers, significant-figures-aware answers, and formula/calculation questions with structured final answers.

Submissions store attempts, answer values, grading results, points earned, feedback shown, and the exact question version used. Versioning preserves what each student saw even if a question is later edited.

## Gradebook

The gradebook shows assignment scores, attempt status, missing and late indicators, and item-level performance. It supports CSV roster import and CSV grade export. LMS integration and grade passback are deferred.

## Technical Direction

The recommended first build is a hosted full-stack web application using Next.js with TypeScript, PostgreSQL, and Prisma or Drizzle. Authentication uses email/password accounts and course invite codes. Hosting should target a managed cloud platform such as Render, Railway, Vercel with managed Postgres, or Azure App Service.

Core modules include `content-ingestion`, `question-bank`, `validation`, `assignments`, `grading`, `practice`, and `admin`. AI may draft questions and explanations, but graded homework must be evaluated by deterministic grading rules, stored answer keys, accepted units, and versioned tolerances.

## Data Model

Initial data models include `User`, `Course`, `Enrollment`, `SourceDocument`, `SourceSection`, `LearningObjective`, `Question`, `QuestionVersion`, `ValidationRun`, `Assignment`, `AssignmentQuestion`, `Submission`, `SubmissionAnswer`, and `GradebookExport`.

## Testing And Reliability

Reliability is a product requirement. The test suite must cover grading rules, tolerance handling, unit conversion, significant figures, answer normalization, assignment submission, grade export, and role-based permissions. Every corrected grading or content bug should receive a regression test.

The platform must keep audit trails for source alignment, validation checks, question approval, reviewer identity, and question versions used in student submissions.

## Rollout

Rollout should happen in phases:

1. Internal prototype: source ingestion, draft question bank, and validation reports.
2. Instructor alpha: review workflow, approvals, course setup, and assignment creation.
3. Student pilot: one course, limited chapters, monitored homework.
4. Expanded use: more chapters, CSV workflows, and adaptive practice.
5. Future integrations: LMS/LTI, departmental reviewer role, and richer response types.

## Operational Requirements

Production deployments should use environment variables for secrets, scheduled database backups, error monitoring, and a kill switch for AI-assisted practice if validation quality drops below acceptable standards.
