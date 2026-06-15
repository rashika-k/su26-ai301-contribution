# su26-ai301-contribution
Github Contribution Log

# Contribution 1: Setting config variable with incompatible type produces unclear error message

**Contribution Number:** 1 

**Student:** Rashika Karmacharya  
**Issue:** https://github.com/mcgill-courses/mcgill.courses/issues/771  
**Status:** Phase 1 Complete

---

## Why I Chose This Issue
I chose this issue because it sits at the intersection of Python and databases, which maps directly to work I've done before. I also wanted a first OSS contribution that was bounded enough to complete well within two weeks but still required real investigation not just a typo or doc update. This issue requires understanding how a completely new language Rust works and tracing where the database needs change. That's a real bug with a real fix, and the kind of thing that's easy to verify once you've solved it.
---

## Understanding the Issue

### Problem Description
Issue #771 requests that the Review type be extended to hold an optional term field (e.g. "Fall 2025", "Winter 2026"), and that the add/edit review form allow users to set it. Currently there is no structured way to record which academic term a reviewer took the course. If they want to share that context, they have to write it manually inside the review body text.
  
### Expected Behavior

When submitting or editing a review, a user should be able to select the term they took the course from a dropdown (e.g. Fall 2025, Winter 2026). That term should be saved with the review and displayed on the review card alongside the instructor name.

### Current Behavior
The Review data type has no term field. The add/edit review form has no term input. Reviews are stored and displayed without any term metadata. The only way to communicate term context is to write it inside the free-text review body.

### Affected Components

  - crates/model/src/review.rs — core Review struct (data model)
  - src/reviews.rs — AddOrUpdateReviewBody request struct and add_review / update_review handlers
  - client/src/lib/types.ts — auto-generated TypeScript types (regenerated via just typeshare)
  - client/src/components/review-form.tsx — shared form used by both add and edit flows
  - client/src/components/add-review-form.tsx — sets initial form values for new reviews
  - client/src/components/edit-review-form.tsx — sets initial form values when editing an existing review
  - client/src/components/course-review.tsx — renders the review card that users see

## Reproduction Process

### Environment Setup

  The project is a Rust + React monorepo. The backend requires a stable Rust toolchain and a running MongoDB instance (via Docker using docker-compose.yml). The frontend uses Node and pnpm. The project uses just as a task runner where running just dev starts both services, and just typeshare regenerates TypeScript types from Rust structs after any model changes.

  Authentication is handled via McGill's Azure AD tenant, which is hardcoded in src/auth.rs (MCGILL_TENANT_ID). Since I don't have a McGill Microsoft account, login  was not possible without modifying that constant locally. Most of the work for this issue — model changes, backend changes, and form UI — is verifiable without being logged in, since course reviews are publicly visible and the form renders regardless of auth state.
  
### Steps to Reproduce

  1. Run the app locally with just dev
  2. Navigate to any course page (e.g. /course/COMP-202)
  3. Click "Add Review" to open the review modal
  4. Observe that the form contains fields for instructor(s), rating, difficulty, and review content — but no field for the term the course was taken
  5. Submit a review and view it on the course page — no term is shown on the review card; the only way to
  communicate term context is writing it inside the review body

### Reproduction Evidence

- **Commit showing reproduction:** [[Link to commit in your fork]](https://github.com/mcgill-courses/mcgill.courses/commit/c04435966450c7ab80beacc5001b599929ba8719)
  
- **Screenshots/logs:** <img width="1102" height="1162" alt="Screenshot 2026-06-15 at 1 37 06 AM" src="https://github.com/user-attachments/assets/d51275f5-739c-4543-a721-b1d810285f0f" />
<img width="1102" height="1162" alt="Screenshot 2026-06-15 at 1 37 13 AM" src="https://github.com/user-attachments/assets/99ea6452-c6bc-4b96-a7f9-aa0f752b7252" />

- **My findings:** The Review struct in crates/model/src/review.rs confirms no term field exists. The AddOrUpdateReviewBody in src/reviews.rs equally has no term. The ReviewForm component in review-form.tsx has inputs for instructors, rating, difficulty, and content only. The CourseReview display component in course-review.tsx renders the instructor line but has no place to show a term.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

