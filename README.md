# su26-ai301-contribution
Github Contribution Log

# Contribution 1: Setting config variable with incompatible type produces unclear error message

**Contribution Number:** 1 

**Student:** Rashika Karmacharya  
**Issue:** https://github.com/mcgill-courses/mcgill.courses/issues/771  
**Status:** Phase 3 In Progress still

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

The Review struct in crates/model/src/review.rs has no term field, so MongoDB never stores one. The request body handler, the frontend form, and the review card display component all follow from the model and none of them have term support either. The fix needs to thread the field through every layer of the stack.

### Proposed Solution

Add term: Option<String> to the Rust Review struct, propagate it through the request body and handlers, regenerate the TypeScript types via typeshare, then add a term selector to the review form and display it on the review card.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Reviews have no way to record which academic term they were written for. Users have to write it in the review body text if they want to communicate it at all.

**Match:** The instructors field is a good reference pattern — it's an optional multi-value field on Review that flows through the model, request body, form, and display card. The term field follows the same path but is a single optional string.

**Plan:** [Step-by-step implementation plan]
1. Add term: Option<String> to Review struct in crates/model/src/review.rs
2. Update Default and Into<Bson> impls
3. Add term to AddOrUpdateReviewBody in src/reviews.rs
4. Run typeshare to regenerate client/src/lib/types.ts
5. Add term selector dropdown to client/src/components/review-form.tsx
6. Update add-review-form.tsx and edit-review-form.tsx initial values
7. Display term on the review card in course-review.tsx
   
**Implement:** [Branch: feat/771-Add-terms-to-course-reviews](https://github.com/rashika-k/mcgill.courses/tree/feat/771-Add-terms-to-course-reviews)

**Review:** No unnecessary changes, tests pass, typeshare regenerated, style matches existing code

**Evaluate:** Run the app locally, open a course page, submit a review with a term selected, confirm it appears on the review card

---

## Testing Strategy

### Unit Tests

#### serialize_term_when_present
Test case 1: verifies that Some("Fall 2025") serializes to the correct JSON string

#### serialize_term_when_absent
Test case 2: verifies that None serializes to JSON null

-  All 24 existing tests still pass with no regressions

### Integration Tests

No integration tests yet

- [Done when I Work on the frontend] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing
cargo build passes cleanly, cargo test -p model 24/24

---

## Implementation Notes

### Week 1 Progress

I worked on the backend this time.
- Added term tp the Review Struct on the review rust file and updated the Default and Into<Bson> as well.
- Ran typeshare to generate a ts file

- Files modified:
  -- crates/model/src/review.rs
  -- client/src/lib/types.ts (auto-generated)
  
### Code Changes

- **Files modified:**
  -- crates/model/src/review.rs
  -- client/src/lib/types.ts (auto-generated)
  
- **Key commits:**
-   [8457125fab45ee051d9300e8f257c16cc95efade](https://github.com/rashika-k/mcgill.courses/commit/8457125fab45ee051d9300e8f257c16cc95efade)
-   [6bf2de529705a4da80bc7fba52073a78af28597e](https://github.com/rashika-k/mcgill.courses/commit/6bf2de529705a4da80bc7fba52073a78af28597e)
  
- **Approach decisions:** Most logical step

---

## Pull Request

**PR Link:** [PR URL](https://github.com/mcgill-courses/mcgill.courses/pull/1170)

**PR Description:** Added an optional `term` field (e.g. "Fall 2025") to reviews so users can record which term they took a course, instead of writing it into the free-text review body. This required extending the `Review` model and `AddOrUpdateReviewBody` on the backend, regenerating the typeshare-generated TypeScript types, and adding a term dropdown to the add/edit review form on the frontend, populated from the course's offered terms merged with a recent-terms range and defaulting to the current term.


**Maintainer Feedback:**
- Not yet received — PR just opened.

**Status:** Awaiting review

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

