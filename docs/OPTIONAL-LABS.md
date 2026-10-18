# Optional advanced labs — AEM 6.5 LTS SP2

These are **post-workshop extensions**, not part of the 11-hour core agenda. Complete Labs 0–15 first. The existing course cards are component instances, **not** `cq:Page` nodes: create course detail pages before trying page searches. Confirm each UI/API on the actual SP2 instance; the exercises below are instructor candidates, not live-instance certified.

## E1 — Query Builder: search course pages (40 minutes, recommended)

**Runnable smoke check:** after creating two real child pages and setting `AEM_PASS`, run `training-portal/scripts/query-builder-smoke.sh`. This prints hit paths for offsets 0 and 1; confirm they differ. The script targets author only and is not a publish endpoint.

**Goal:** find authored course detail pages without traversing the entire repository. **Prerequisite:** Lab 3 and Lab 11's two course detail child pages at `/content/trainingportal/us/en/courses/` (for example `aem-fundamentals` and `component-development`). Set `jcr:title` and `jcr:description` on their `jcr:content` nodes.

1. Open the Query Builder debugger at `/libs/cq/search/content/querydebug.html` on **author only** (check the URL exists on SP2). Select JSON output and submit these predicates:

   ```properties
   path=/content/trainingportal/us/en/courses
   type=cq:Page
   1_property=jcr:content/jcr:title
   1_property.operation=exists
   p.limit=10
   p.offset=0
   ```

2. Confirm that returned paths are **pages**, not the Course Card component nodes. Try `p.limit=1` and `p.offset=1` and confirm pagination. Add `orderby=@jcr:content/jcr:title` and `orderby.sort=asc` for a stable order. Check whether the order-by syntax works on the installed build.
3. Add a title filter using `1_property.value=AEM Fundamentals`; compare exact matching with full-text search. Add a `tagid` predicate **only after** assigning a real tag to the course pages.
4. Review the query in Query Performance/Explain Query tools (where available). Look for traversals and index warnings. **Do not add an Oak index blindly**: index design and deployment require an environment-specific review.
5. **Checkpoint:** at least two page results before pagination; page 2 returns a different page; no results outside the course subtree. Document actual hit counts, query time and the chosen index.

**Security:** never expose the Query Builder debugger on publish. Repository ACLs and Dispatcher rules still apply to any public search endpoint. Cap page size and validate paths if you implement a servlet.

## E2 — Workflow approval routing (35 minutes)

**Goal:** extend Lab 12's validator to make its result actionable. **Prerequisite:** Lab 12 and a course page with `jcr:title` and `jcr:description` on `jcr:content`.

1. Clone the existing Training Portal Course Approval model (keep the original as a known-good fallback).
2. Add a split after the `Training Portal Course Validation` process. Route `validationResult=valid` to a **Review course** participant step and `invalid` to a **Fix metadata** participant step.
3. In each branch, complete the Inbox task as the assigned user. Resubmit an invalid page after adding the missing description.
4. Sync the model, start one valid and one invalid instance, and inspect the workflow metadata/history in the UI and repository.
5. **Checkpoint:** valid and invalid payloads take different branches; no automatic activation; all instances reach a terminal state after user completion.

**Important:** ECMA/script bindings and split rule evaluation differ by build. Inspect the rule editor on your SP2 instance and test with two payloads before giving learners a copy-paste script. Do not use workflow metadata as a substitute for content-level validation on activation.

## E3 — Core Component proxy and Sling Model delegation (35 minutes)

**Goal:** compare a custom HTL component with a supported proxy component.

1. Inspect the installed Core Components version. Create `/apps/trainingportal/components/course-title` with `sling:resourceSuperType` pointing to the **installed** Core Title component's supported resource type.
2. Add it to the editable template's allowed components policy; author a Title in the Courses page.
3. Demonstrate how a proxy inherits markup and dialog behavior. Only then sketch a Sling Model delegation pattern with `@Via(type=ResourceSuperType.class)` and an interface matching the installed Core Component API; avoid hard-coding a version until verified.
4. **Checkpoint:** Title renders in author and `wcmmode=disabled`, and no Adobe `/libs` code is copied into the project.

## E4 — Layered service architecture and unit testing (35 minutes)

**Goal:** refactor a feature into `Sling Servlet -> CourseSearchService -> Repository adapter` without putting query construction in the servlet.

1. Create a `CourseSearchService` interface accepting a validated site path, a search term and a bounded page size (1–20).
2. Implement the repository adapter using the Query Builder predicates from E1, or a controlled ResourceResolver-based lookup for a tiny fixed dataset. Do not use repository-wide traversal.
3. Inject the service with OSGi DS `@Reference` in a resource-type-bound GET servlet. Return a JSON object with `items` and `count`; never expose JCR internals or raw exception messages.
4. Unit-test invalid path, blank query, max page size, empty results and service failures. Mock the service in servlet tests.
5. **Checkpoint:** requests cannot search outside `/content/trainingportal`, and the service returns a deterministic result for the fixture pages.

## E5 — Content Fragments (40 minutes, optional if enabled)

Create a Course Content Fragment Model under the project configuration, author two fragments, and compare structured content with page/component content. Inspect which GraphQL or other headless endpoints are **actually supported and configured** on your SP2 instance before writing API instructions. Do not assume AEM as a Cloud Service endpoint conventions apply to LTS.

## E6 — Resource Resolver, permissions and indexing (25 minutes)

Inspect the Lab 11 read-only service user, demonstrate `getServiceResourceResolver` and try-with-resources, and verify that a read is allowed but a write is denied. Compare `Resource`, `ValueMap`, and `Session` adaptation. Explain why Query Builder does not eliminate the need for index analysis or ACL testing.

## Recommended delivery choices

| Scenario | Mandatory | Optional |
|---|---|---|
| 10–11 hour class | Existing Labs 0–15 | Instructor demo: 10-minute Query Builder debugger teaser, using Lab 11 pages |
| Extra 45 minutes | Existing Labs 0–15 | E1 Query Builder |
| Extra 90 minutes | Existing Labs 0–15 | E1 Query Builder + E2 Workflow routing |
| Follow-up half-day | Existing Labs 0–15 already completed | E1–E4; E5/E6 as take-home |

## Instructor acceptance checks

- Verify the exact SP2 instance, Java version, Core Components version and Query Builder debugger URL.
- Use **two actual course pages** as E1 fixtures; do not count Course Card component instances as pages.
- Record E1's expected hit count and whether pagination/order are stable on a clean instance.
- Run valid and invalid E2 workflows and verify the Inbox assignee and completed history.
- Confirm E3's resource super type exists in the installed Core Components release.
- Run unit tests and perform a least-privilege ACL test for E4/E6.
- Mark any feature not exercised on the actual instance as *demonstration only*, not verified.
