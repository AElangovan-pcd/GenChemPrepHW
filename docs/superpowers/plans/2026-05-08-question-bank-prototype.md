# Question Bank Prototype Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the first independently testable GenChemPrepHW slice: a Next.js TypeScript app with deterministic chemistry grading utilities, source-section records, question versions, validation runs, and an instructor-facing review workflow.

**Architecture:** The first milestone centers on the reviewed question bank. AI drafting is represented by stored draft records; graded delivery is blocked until deterministic validation and instructor approval are implemented. Assignments, submissions, CSV gradebook, and LMS integration are handled in separate follow-up plans.

**Tech Stack:** Next.js App Router, TypeScript, PostgreSQL, Prisma, Vitest, React Testing Library, ESLint, Prettier.

---

## File Structure

- Create `package.json`: scripts and dependencies.
- Create `tsconfig.json`: strict TypeScript config.
- Create `next.config.ts`: Next.js configuration.
- Create `eslint.config.mjs`: lint configuration.
- Create `vitest.config.ts`: unit test configuration.
- Create `src/app/page.tsx`: instructor review dashboard entry point.
- Create `src/app/layout.tsx`: application shell.
- Create `src/app/globals.css`: basic app styling.
- Create `src/lib/chemistry/normalize.ts`: numeric, unit, and significant-figure helpers.
- Create `src/lib/chemistry/grading.ts`: deterministic grading functions.
- Create `src/lib/chemistry/grading.test.ts`: grading behavior tests.
- Create `src/lib/question-bank/types.ts`: shared question-bank types.
- Create `src/lib/question-bank/workflow.ts`: state transition rules.
- Create `src/lib/question-bank/workflow.test.ts`: approval-state tests.
- Create `src/lib/validation/validation.ts`: validation report builder.
- Create `src/lib/validation/validation.test.ts`: validation tests.
- Create `prisma/schema.prisma`: first database schema.
- Create `.env.example`: required environment variables.

---

### Task 1: Scaffold The Next.js TypeScript Project

**Files:**
- Create: `package.json`
- Create: `tsconfig.json`
- Create: `next.config.ts`
- Create: `eslint.config.mjs`
- Create: `vitest.config.ts`
- Create: `src/app/layout.tsx`
- Create: `src/app/page.tsx`
- Create: `src/app/globals.css`

- [ ] **Step 1: Create package metadata and scripts**

Write `package.json`:

```json
{
  "name": "genchemprephw",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "eslint .",
    "test": "vitest run",
    "test:watch": "vitest"
  },
  "dependencies": {
    "@prisma/client": "^5.22.0",
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "zod": "^3.23.8"
  },
  "devDependencies": {
    "@eslint/js": "^9.15.0",
    "@testing-library/jest-dom": "^6.6.3",
    "@testing-library/react": "^16.0.1",
    "@types/node": "^22.9.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "eslint": "^9.15.0",
    "eslint-config-next": "^15.0.0",
    "jsdom": "^25.0.1",
    "prettier": "^3.3.3",
    "prisma": "^5.22.0",
    "typescript": "^5.6.3",
    "vitest": "^2.1.5"
  }
}
```

- [ ] **Step 2: Create TypeScript and tool configuration**

Write `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["dom", "dom.iterable", "es2022"],
    "allowJs": false,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

Write `next.config.ts`:

```ts
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true
};

export default nextConfig;
```

Write `eslint.config.mjs`:

```js
import js from "@eslint/js";
import nextPlugin from "eslint-config-next";

export default [
  js.configs.recommended,
  ...nextPlugin,
  {
    ignores: [".next/**", "node_modules/**", "coverage/**"]
  }
];
```

Write `vitest.config.ts`:

```ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
    globals: true,
    include: ["src/**/*.test.ts", "src/**/*.test.tsx"]
  },
  resolve: {
    alias: {
      "@": new URL("./src", import.meta.url).pathname
    }
  }
});
```

- [ ] **Step 3: Create the initial app shell**

Write `src/app/layout.tsx`:

```tsx
import "./globals.css";
import type { Metadata } from "next";
import type { ReactNode } from "react";

export const metadata: Metadata = {
  title: "GenChemPrepHW",
  description: "Reviewed general chemistry homework platform"
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

Write `src/app/page.tsx`:

```tsx
export default function HomePage() {
  return (
    <main className="page">
      <section className="hero">
        <p className="eyebrow">GenChemPrepHW</p>
        <h1>Reviewed question bank for college general chemistry</h1>
        <p>
          Draft questions stay out of graded homework until validation and
          instructor approval are complete.
        </p>
      </section>
    </main>
  );
}
```

Write `src/app/globals.css`:

```css
:root {
  color-scheme: light;
  font-family:
    Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI",
    sans-serif;
  background: #f7f7f4;
  color: #18201f;
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

.page {
  min-height: 100vh;
  padding: 48px;
}

.hero {
  max-width: 760px;
}

.eyebrow {
  margin: 0 0 12px;
  color: #276461;
  font-weight: 700;
  text-transform: uppercase;
}

h1 {
  margin: 0 0 16px;
  font-size: 44px;
  line-height: 1.05;
}

p {
  font-size: 18px;
  line-height: 1.6;
}
```

- [ ] **Step 4: Install dependencies**

Run: `npm install`

Expected: dependencies install and `package-lock.json` is created.

- [ ] **Step 5: Verify scaffold**

Run: `npm test`

Expected: Vitest exits successfully with no tests found only if Vitest is configured to pass with an empty suite; if it fails because no tests exist, continue to Task 2 before making the first commit.

Run: `npm run build`

Expected: Next.js builds the initial page successfully.

- [ ] **Step 6: Commit scaffold**

```bash
git add package.json package-lock.json tsconfig.json next.config.ts eslint.config.mjs vitest.config.ts src/app
git commit -m "feat: scaffold GenChemPrepHW app"
```

---

### Task 2: Implement Deterministic Chemistry Grading Utilities

**Files:**
- Create: `src/lib/chemistry/normalize.ts`
- Create: `src/lib/chemistry/grading.ts`
- Create: `src/lib/chemistry/grading.test.ts`

- [ ] **Step 1: Write failing grading tests**

Write `src/lib/chemistry/grading.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import {
  gradeNumericAnswer,
  gradeSignificantFigures,
  gradeUnitsAnswer
} from "./grading";

describe("chemistry grading", () => {
  it("accepts numeric answers inside tolerance", () => {
    expect(
      gradeNumericAnswer({ submitted: "10.01", expected: 10, tolerance: 0.02 })
    ).toEqual({ correct: true, normalizedSubmitted: 10.01 });
  });

  it("rejects numeric answers outside tolerance", () => {
    expect(
      gradeNumericAnswer({ submitted: "10.05", expected: 10, tolerance: 0.02 })
        .correct
    ).toBe(false);
  });

  it("checks required significant figures", () => {
    expect(gradeSignificantFigures({ submitted: "12.30", required: 4 })).toEqual({
      correct: true,
      submittedFigures: 4
    });
  });

  it("accepts compatible simple metric units", () => {
    expect(
      gradeUnitsAnswer({
        submittedValue: "1000",
        submittedUnit: "mL",
        expectedValue: 1,
        expectedUnit: "L",
        tolerance: 0.0001
      }).correct
    ).toBe(true);
  });
});
```

- [ ] **Step 2: Run tests to verify failure**

Run: `npm test -- src/lib/chemistry/grading.test.ts`

Expected: FAIL because `./grading` does not exist.

- [ ] **Step 3: Implement normalization helpers**

Write `src/lib/chemistry/normalize.ts`:

```ts
const unitFactors: Record<string, number> = {
  L: 1,
  mL: 0.001,
  g: 1,
  kg: 1000,
  mol: 1,
  mmol: 0.001
};

export function parseFiniteNumber(value: string): number | null {
  const normalized = Number(value.trim());
  return Number.isFinite(normalized) ? normalized : null;
}

export function countSignificantFigures(value: string): number {
  const cleaned = value.trim().replace(/^[-+]/, "");
  if (cleaned.includes(".")) {
    const withoutDecimal = cleaned.replace(".", "");
    const withoutLeadingZeros = withoutDecimal.replace(/^0+/, "");
    return withoutLeadingZeros.length;
  }

  const withoutLeadingZeros = cleaned.replace(/^0+/, "");
  const withoutTrailingZeros = withoutLeadingZeros.replace(/0+$/, "");
  return withoutTrailingZeros.length;
}

export function convertUnit(value: number, fromUnit: string, toUnit: string): number | null {
  const fromFactor = unitFactors[fromUnit];
  const toFactor = unitFactors[toUnit];

  if (fromFactor === undefined || toFactor === undefined) {
    return null;
  }

  return (value * fromFactor) / toFactor;
}
```

- [ ] **Step 4: Implement grading functions**

Write `src/lib/chemistry/grading.ts`:

```ts
import {
  convertUnit,
  countSignificantFigures,
  parseFiniteNumber
} from "./normalize";

export type NumericGrade = {
  correct: boolean;
  normalizedSubmitted: number | null;
};

export function gradeNumericAnswer(input: {
  submitted: string;
  expected: number;
  tolerance: number;
}): NumericGrade {
  const normalizedSubmitted = parseFiniteNumber(input.submitted);

  if (normalizedSubmitted === null) {
    return { correct: false, normalizedSubmitted };
  }

  return {
    correct: Math.abs(normalizedSubmitted - input.expected) <= input.tolerance,
    normalizedSubmitted
  };
}

export function gradeSignificantFigures(input: {
  submitted: string;
  required: number;
}): { correct: boolean; submittedFigures: number } {
  const submittedFigures = countSignificantFigures(input.submitted);

  return {
    correct: submittedFigures === input.required,
    submittedFigures
  };
}

export function gradeUnitsAnswer(input: {
  submittedValue: string;
  submittedUnit: string;
  expectedValue: number;
  expectedUnit: string;
  tolerance: number;
}): NumericGrade {
  const numericValue = parseFiniteNumber(input.submittedValue);

  if (numericValue === null) {
    return { correct: false, normalizedSubmitted: null };
  }

  const converted = convertUnit(numericValue, input.submittedUnit, input.expectedUnit);

  if (converted === null) {
    return { correct: false, normalizedSubmitted: null };
  }

  return {
    correct: Math.abs(converted - input.expectedValue) <= input.tolerance,
    normalizedSubmitted: converted
  };
}
```

- [ ] **Step 5: Run tests**

Run: `npm test -- src/lib/chemistry/grading.test.ts`

Expected: PASS for all four grading tests.

- [ ] **Step 6: Commit grading utilities**

```bash
git add src/lib/chemistry
git commit -m "feat: add deterministic chemistry grading utilities"
```

---

### Task 3: Add Question Bank Types And Approval Workflow

**Files:**
- Create: `src/lib/question-bank/types.ts`
- Create: `src/lib/question-bank/workflow.ts`
- Create: `src/lib/question-bank/workflow.test.ts`

- [ ] **Step 1: Write failing workflow tests**

Write `src/lib/question-bank/workflow.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { canUseInGradedAssignment, nextQuestionState } from "./workflow";

describe("question-bank workflow", () => {
  it("blocks draft questions from graded assignments", () => {
    expect(canUseInGradedAssignment("draft")).toBe(false);
  });

  it("blocks validated questions from graded assignments", () => {
    expect(canUseInGradedAssignment("validated")).toBe(false);
  });

  it("allows approved questions in graded assignments", () => {
    expect(canUseInGradedAssignment("approved")).toBe(true);
  });

  it("moves validated questions to approved after instructor approval", () => {
    expect(nextQuestionState("validated", "approve")).toBe("approved");
  });
});
```

- [ ] **Step 2: Run tests to verify failure**

Run: `npm test -- src/lib/question-bank/workflow.test.ts`

Expected: FAIL because `./workflow` does not exist.

- [ ] **Step 3: Add shared question-bank types**

Write `src/lib/question-bank/types.ts`:

```ts
export type QuestionState = "draft" | "validated" | "approved" | "rejected";

export type QuestionType =
  | "multiple_choice"
  | "true_false"
  | "matching"
  | "numeric"
  | "units_numeric"
  | "significant_figures"
  | "formula_calculation";

export type SourceAlignment = {
  sourceDocumentId: string;
  sourceSectionId: string;
  sourceUrl: string;
  learningObjective: string;
};

export type GradingRule = {
  type: QuestionType;
  expectedAnswer: string;
  tolerance?: number;
  expectedUnit?: string;
  requiredSignificantFigures?: number;
};

export type QuestionVersion = {
  id: string;
  questionId: string;
  prompt: string;
  workedSolution: string;
  gradingRule: GradingRule;
  sourceAlignment: SourceAlignment;
  createdAt: string;
};
```

- [ ] **Step 4: Implement workflow rules**

Write `src/lib/question-bank/workflow.ts`:

```ts
import type { QuestionState } from "./types";

export type ReviewAction = "validate" | "approve" | "reject" | "revise";

export function canUseInGradedAssignment(state: QuestionState): boolean {
  return state === "approved";
}

export function nextQuestionState(
  currentState: QuestionState,
  action: ReviewAction
): QuestionState {
  if (action === "reject") {
    return "rejected";
  }

  if (action === "revise") {
    return "draft";
  }

  if (currentState === "draft" && action === "validate") {
    return "validated";
  }

  if (currentState === "validated" && action === "approve") {
    return "approved";
  }

  return currentState;
}
```

- [ ] **Step 5: Run tests**

Run: `npm test -- src/lib/question-bank/workflow.test.ts`

Expected: PASS for all workflow tests.

- [ ] **Step 6: Commit workflow rules**

```bash
git add src/lib/question-bank
git commit -m "feat: add question approval workflow"
```

---

### Task 4: Add Validation Report Builder

**Files:**
- Create: `src/lib/validation/validation.ts`
- Create: `src/lib/validation/validation.test.ts`

- [ ] **Step 1: Write failing validation tests**

Write `src/lib/validation/validation.test.ts`:

```ts
import { describe, expect, it } from "vitest";
import { buildValidationReport } from "./validation";

describe("validation report builder", () => {
  it("passes a numeric question with source alignment and worked solution", () => {
    const report = buildValidationReport({
      sourceUrl: "https://chem-139-oer-text-2026.onrender.com/",
      prompt: "Calculate the molar mass of CO2.",
      expectedAnswer: "44.01",
      workedSolution: "C contributes 12.01 g/mol and two O atoms contribute 32.00 g/mol.",
      gradingRule: { type: "numeric", expectedAnswer: "44.01", tolerance: 0.01 }
    });

    expect(report.status).toBe("passed");
    expect(report.checks.map((check) => check.name)).toContain("source_alignment");
  });

  it("fails a question without source alignment", () => {
    const report = buildValidationReport({
      sourceUrl: "",
      prompt: "Calculate the molar mass of CO2.",
      expectedAnswer: "44.01",
      workedSolution: "C contributes 12.01 g/mol and two O atoms contribute 32.00 g/mol.",
      gradingRule: { type: "numeric", expectedAnswer: "44.01", tolerance: 0.01 }
    });

    expect(report.status).toBe("failed");
  });
});
```

- [ ] **Step 2: Run tests to verify failure**

Run: `npm test -- src/lib/validation/validation.test.ts`

Expected: FAIL because `./validation` does not exist.

- [ ] **Step 3: Implement validation report builder**

Write `src/lib/validation/validation.ts`:

```ts
import type { GradingRule } from "@/lib/question-bank/types";

export type ValidationCheck = {
  name:
    | "source_alignment"
    | "prompt_present"
    | "answer_present"
    | "worked_solution_present"
    | "grading_rule_present";
  passed: boolean;
  message: string;
};

export type ValidationReport = {
  status: "passed" | "failed";
  checks: ValidationCheck[];
};

export function buildValidationReport(input: {
  sourceUrl: string;
  prompt: string;
  expectedAnswer: string;
  workedSolution: string;
  gradingRule: GradingRule;
}): ValidationReport {
  const checks: ValidationCheck[] = [
    {
      name: "source_alignment",
      passed: input.sourceUrl.startsWith("https://"),
      message: "Question must link to an HTTPS source section."
    },
    {
      name: "prompt_present",
      passed: input.prompt.trim().length >= 10,
      message: "Question prompt must be specific enough for review."
    },
    {
      name: "answer_present",
      passed: input.expectedAnswer.trim().length > 0,
      message: "Question must include an expected answer."
    },
    {
      name: "worked_solution_present",
      passed: input.workedSolution.trim().length >= 20,
      message: "Question must include a reviewable worked solution."
    },
    {
      name: "grading_rule_present",
      passed: input.gradingRule.expectedAnswer.trim().length > 0,
      message: "Question must include deterministic grading metadata."
    }
  ];

  return {
    status: checks.every((check) => check.passed) ? "passed" : "failed",
    checks
  };
}
```

- [ ] **Step 4: Run validation tests**

Run: `npm test -- src/lib/validation/validation.test.ts`

Expected: PASS for both validation tests.

- [ ] **Step 5: Commit validation report builder**

```bash
git add src/lib/validation
git commit -m "feat: add question validation reports"
```

---

### Task 5: Add Initial Prisma Data Model

**Files:**
- Create: `prisma/schema.prisma`
- Create: `.env.example`

- [ ] **Step 1: Create environment example**

Write `.env.example`:

```bash
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/genchemprephw"
NEXTAUTH_SECRET="replace-with-a-strong-secret"
```

- [ ] **Step 2: Create Prisma schema**

Write `prisma/schema.prisma`:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  ADMIN
  INSTRUCTOR
  STUDENT
}

enum QuestionState {
  DRAFT
  VALIDATED
  APPROVED
  REJECTED
}

model User {
  id           String       @id @default(cuid())
  email        String       @unique
  name         String
  role         UserRole
  createdAt    DateTime     @default(now())
  enrollments  Enrollment[]
  approvedRuns ValidationRun[] @relation("ApprovedBy")
}

model Course {
  id          String       @id @default(cuid())
  title       String
  inviteCode  String       @unique
  createdAt   DateTime     @default(now())
  enrollments Enrollment[]
}

model Enrollment {
  id        String   @id @default(cuid())
  userId    String
  courseId  String
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
  course    Course   @relation(fields: [courseId], references: [id])
}

model SourceDocument {
  id        String          @id @default(cuid())
  title     String
  url       String          @unique
  createdAt DateTime        @default(now())
  sections  SourceSection[]
}

model SourceSection {
  id               String            @id @default(cuid())
  sourceDocumentId String
  title            String
  url              String            @unique
  orderIndex       Int
  sourceDocument   SourceDocument    @relation(fields: [sourceDocumentId], references: [id])
  questions        QuestionVersion[]
}

model Question {
  id        String            @id @default(cuid())
  state     QuestionState     @default(DRAFT)
  createdAt DateTime          @default(now())
  versions  QuestionVersion[]
}

model QuestionVersion {
  id              String          @id @default(cuid())
  questionId      String
  sourceSectionId String
  prompt          String
  expectedAnswer  String
  workedSolution  String
  gradingRule     Json
  versionNumber   Int
  createdAt       DateTime        @default(now())
  question        Question        @relation(fields: [questionId], references: [id])
  sourceSection   SourceSection   @relation(fields: [sourceSectionId], references: [id])
  validationRuns  ValidationRun[]
}

model ValidationRun {
  id                String          @id @default(cuid())
  questionVersionId String
  status            String
  checks            Json
  approvedById      String?
  createdAt         DateTime        @default(now())
  questionVersion   QuestionVersion @relation(fields: [questionVersionId], references: [id])
  approvedBy        User?           @relation("ApprovedBy", fields: [approvedById], references: [id])
}
```

- [ ] **Step 3: Generate Prisma client**

Run: `npx prisma generate`

Expected: Prisma Client generation completes successfully.

- [ ] **Step 4: Validate schema**

Run: `npx prisma validate`

Expected: Prisma reports that the schema is valid.

- [ ] **Step 5: Commit data model**

```bash
git add .env.example prisma/schema.prisma package.json package-lock.json
git commit -m "feat: add question bank data model"
```

---

### Task 6: Build Instructor Review Dashboard Prototype

**Files:**
- Modify: `src/app/page.tsx`
- Create: `src/lib/question-bank/sample-data.ts`
- Create: `src/app/page.test.tsx`

- [ ] **Step 1: Write failing dashboard test**

Write `src/app/page.test.tsx`:

```tsx
import "@testing-library/jest-dom/vitest";
import { render, screen } from "@testing-library/react";
import { describe, expect, it } from "vitest";
import HomePage from "./page";

describe("HomePage", () => {
  it("shows draft, validated, and approved question counts", () => {
    render(<HomePage />);

    expect(screen.getByText("Draft")).toBeInTheDocument();
    expect(screen.getByText("Validated")).toBeInTheDocument();
    expect(screen.getByText("Approved")).toBeInTheDocument();
  });
});
```

- [ ] **Step 2: Run test to verify failure**

Run: `npm test -- src/app/page.test.tsx`

Expected: FAIL because the page does not show the status counts.

- [ ] **Step 3: Add sample question-bank records**

Write `src/lib/question-bank/sample-data.ts`:

```ts
import type { QuestionState, QuestionVersion } from "./types";

export type ReviewCard = {
  id: string;
  state: QuestionState;
  title: string;
  version: QuestionVersion;
};

export const reviewCards: ReviewCard[] = [
  {
    id: "q-co2-molar-mass",
    state: "validated",
    title: "Molar mass of carbon dioxide",
    version: {
      id: "qv-co2-molar-mass-1",
      questionId: "q-co2-molar-mass",
      prompt: "Calculate the molar mass of CO2 using standard atomic masses.",
      workedSolution:
        "Carbon contributes 12.01 g/mol and two oxygen atoms contribute 32.00 g/mol, for a total of 44.01 g/mol.",
      gradingRule: {
        type: "numeric",
        expectedAnswer: "44.01",
        tolerance: 0.01
      },
      sourceAlignment: {
        sourceDocumentId: "chem-139-oer",
        sourceSectionId: "moles-molar-mass",
        sourceUrl: "https://chem-139-oer-text-2026.onrender.com/",
        learningObjective: "Calculate molar mass from a chemical formula."
      },
      createdAt: "2026-05-08T00:00:00.000Z"
    }
  }
];
```

- [ ] **Step 4: Update dashboard page**

Replace `src/app/page.tsx` with:

```tsx
import { reviewCards } from "@/lib/question-bank/sample-data";

const states = ["draft", "validated", "approved"] as const;

export default function HomePage() {
  const counts = states.map((state) => ({
    state,
    count: reviewCards.filter((card) => card.state === state).length
  }));

  return (
    <main className="page">
      <section className="hero">
        <p className="eyebrow">GenChemPrepHW</p>
        <h1>Instructor question review</h1>
        <p>
          Review validation results before approving chemistry questions for
          graded assignments.
        </p>
      </section>

      <section className="statusGrid" aria-label="Question status counts">
        {counts.map((item) => (
          <article className="statusCard" key={item.state}>
            <h2>{item.state[0].toUpperCase() + item.state.slice(1)}</h2>
            <p>{item.count}</p>
          </article>
        ))}
      </section>

      <section className="reviewList" aria-label="Questions ready for review">
        {reviewCards.map((card) => (
          <article className="questionCard" key={card.id}>
            <p className="eyebrow">{card.state}</p>
            <h2>{card.title}</h2>
            <p>{card.version.prompt}</p>
            <h3>Worked solution</h3>
            <p>{card.version.workedSolution}</p>
            <a href={card.version.sourceAlignment.sourceUrl}>Open source alignment</a>
          </article>
        ))}
      </section>
    </main>
  );
}
```

- [ ] **Step 5: Extend styling**

Append to `src/app/globals.css`:

```css
.statusGrid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 16px;
  margin: 32px 0;
  max-width: 900px;
}

.statusCard,
.questionCard {
  border: 1px solid #d8ddd8;
  border-radius: 8px;
  background: #ffffff;
  padding: 20px;
}

.statusCard h2,
.questionCard h2,
.questionCard h3 {
  margin: 0 0 8px;
}

.statusCard p {
  margin: 0;
  font-size: 32px;
  font-weight: 700;
}

.reviewList {
  display: grid;
  gap: 16px;
  max-width: 900px;
}

a {
  color: #276461;
  font-weight: 700;
}
```

- [ ] **Step 6: Run dashboard test and build**

Run: `npm test -- src/app/page.test.tsx`

Expected: PASS.

Run: `npm run build`

Expected: production build completes successfully.

- [ ] **Step 7: Commit dashboard prototype**

```bash
git add src/app src/lib/question-bank/sample-data.ts
git commit -m "feat: add instructor review dashboard prototype"
```

---

## Self-Review

Spec coverage:

- Reviewed question bank foundation is covered by Tasks 3, 4, 5, and 6.
- Deterministic grading for numeric, units, and significant figures is covered by Task 2.
- Source alignment and validation history are covered by Tasks 4 and 5.
- Instructor review is covered by Task 6.
- Assignments, submissions, CSV gradebook, authentication, and deployed hosting are outside this first milestone and should each receive its own implementation plan.

All tasks in this plan include exact files and concrete content.
