# Instructor validation and release gate — AEM 6.5 LTS SP2

**Status:** Static corrections applied. A full live AEM 6.5 LTS SP2 exercise run has **not** been performed. Do not label this kit fully tested until the evidence below is collected.

## Pre-class preparation

1. Use an isolated author on port 4502 with AEM 6.5 LTS SP2, JDK 21 and the stated Maven archetype. Run `preflight.sh`. Record product info, Core Components version and actual bundle symbolic name.
2. Generate a fresh project using Lab 1, apply `patch-archetype-for-lts.py`, inspect Core Components packaging, then run the full Maven build and deploy.
3. Run Labs 0–7 in order. Check that the editable template allows placement of Header/Footer and that the Style System policy appears on **existing** Course Cards.
4. In Lab 11 create two child **cq:Page** nodes under `/content/trainingportal/us/en/courses` before expecting the scheduler to log 2 pages. Two cards on the Courses page do not count.
5. Before Lab 12 set `jcr:title` and `jcr:description` on `/content/trainingportal/us/en/courses/jcr:content`; test a valid workflow and a separate invalid case. Confirm model Sync, Inbox completion and Archive history.
6. For publish/Dispatcher assertions, provision and test a separate publish tier and Dispatcher; author-only tests cannot establish Dispatcher security.

## Required evidence checklist

- [ ] `mvn clean install` succeeds on the generated LTS-patched project; all unit tests pass.
- [ ] `all` package installs; core bundle and OSGi components are Active.
- [ ] Course Page template and home/courses/about pages render; header and footer show on all pages.
- [ ] Two Course Cards persist and render; Sling Model boundary cases and JSON export pass.
- [ ] CSS and JS load through `/etc.clientlibs`; Featured and Compact styles apply to existing cards.
- [ ] Config changes affect the servlet; unauthenticated request never reveals diagnostic JSON.
- [ ] DAM image renders, and missing/foreign paths are handled.
- [ ] Service user mapping and read-only permissions verified; scheduler log shows **2 child pages** after setup.
- [ ] Workflow valid and invalid outcomes verified, participant step completed, history archived.
- [ ] Content package exported and restored on a clean compatible snapshot.
- [ ] Publish/Dispatcher security tested if included in the delivered workshop.

## Scope limitations

`verify.sh all` provides smoke tests for selected labs; it cannot validate visual UI steps, workflow history, accessibility or Dispatcher behavior. Lab 14 Dispatcher snippet is explanatory only. The archive includes lab overlays, not an entire Maven project; participants generate that project in Lab 1.

## Optional exercises

Before teaching extensions, follow the acceptance checks in [OPTIONAL-LABS.md](OPTIONAL-LABS.md). They are not part of the core 11-hour sign-off.
