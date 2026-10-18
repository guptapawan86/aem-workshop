# Part A: About This Guide

## A1. Overview

The **Training Portal Lab** is a hands-on developer course for **Adobe Experience Manager 6.5 LTS (Service Pack 2)** running on **Java 21**. Over **16 labs** and **11 hours** (two days of 5½ hours), participants build one complete, working site and, along the way, practise the core skills of an AEM developer. Every lab starts on a new page, so it can be printed or handed out on its own.

**Who it is for:** developers who know Java and basic web development and are new to AEM, or who are moving to 6.5 LTS from earlier versions.

**By the end, participants can:**
- generate, build and deploy an AEM project with Maven, and adapt it to the LTS API jar;
- create an editable template, policies and pages, and build components with dialogs and HTL;
- write Sling Models, OSGi services, servlets, schedulers and workflow steps, with unit tests;
- package client libraries, apply the Style System, and use the DAM;
- run background code as a least-privilege service user and apply a security checklist;
- diagnose problems with the platform's consoles and logs.

**What they build:** a small "Training Portal" site with a header and footer, a **Course Card** component (dialog, Sling Model, styles, DAM image), a configurable course service, a JSON servlet, a scheduled job and a course-validation workflow.

| Part | Content |
|---|---|
| A | Overview, platform facts, prerequisites, agenda |
| B | Day 1, Labs 0 to 7: project, template, header/footer, Course Card, Sling Model, client libraries |
| C | Day 2, Labs 8 to 14: OSGi service, servlet, DAM, scheduler, workflow, testing, packaging and security |
| D | Lab 15 assessment and appendices |

The companion **lab kit** (`trainingportal-lts-lab-kit.zip`, also in the repository) contains the finished code for every lab in the layout of the generated project, plus helper scripts: `preflight.sh` (environment check), `verify.sh` (a pass/fail check for each lab), `apply-module.sh` (copies a lab's finished files into your project) and `patch-archetype-for-lts.py` (the LTS adjustment used in Lab 1).

---

## A2. Platform facts for AEM 6.5 LTS {: .flow}

| Fact | Consequence in the labs |
|---|---|
| AEM 6.5 LTS is internally versioned "6.6". Java 17 and Java 21 are supported. | The labs use Java 21. Check your service pack at `/system/console/status-productinfo`. |
| The API jar is `com.adobe.aem:uber-jar:6.6.0` with classifier **`apis`**. | Lab 1 applies this to the generated project. Without the classifier you get an empty stub jar and "package does not exist" errors. |
| LTS uses **`javax.*`** namespaces, not `jakarta.*`. | Use `javax.annotation.PostConstruct`. A `jakarta.annotation` import is silently ignored by Sling Models. |
| Google Guava is no longer bundled. | Do not depend on it; bring your own version if you need it. |
| Service packs ship their own Core Components. | Do not embed an older copy over the platform's (Lab 1, step 4). |

## A3. Prerequisites {: .flow}

**Instructor**
- [ ] An author instance on `localhost:4502` at **Service Pack 2** for every participant (one each is best).
- [ ] JDK 21, Maven 3.9+, Git, an editor (IntelliJ IDEA or VS Code) and Bash (Git Bash or WSL on Windows).
- [ ] A warm Maven cache: run the full build once, then share `~/.m2/repository`. The first build also downloads Node and npm packages for `ui.frontend`.
- [ ] If `uber-jar:6.6.0` does not resolve, add your organisation's Adobe Maven repository profile to `~/.m2/settings.xml`.
- [ ] One licence-free **JPG** (about 1200 by 800) for Lab 10.
- [ ] The administrator password, exported as `AEM_PASS` in your own shell. **Never commit it.**

**Each participant**
- [ ] `java -version` shows 21; `mvn -version` and `git --version` work.
- [ ] Can log in to their author instance as an administrator.

## A4. Pre-class validation (90 minutes, the day before) {: .flow}

Run the labs once on a clean instance or a snapshot so the day itself runs smoothly. Note any differences for your own handout: console URLs and a few repository paths can vary slightly between builds.

| # | Do | Expected |
|---|---|---|
| 1 | `preflight.sh` | tools OK; service pack and Core Components versions printed; console URLs reachable |
| 2 | Lab 1: generate, patch, build, deploy | `BUILD SUCCESS`; core bundle **Active** |
| 3 | Labs 3 and 4 in the UI | template enabled; header and footer on every page |
| 4 | `apply-module.sh all <project>`, then one full build | `BUILD SUCCESS`, all unit tests green |
| 5 | `verify.sh all` | no FAIL |
| 6 | Lab 12 end to end | model synced, workflow completed |
| 7 | Lab 14: build the **golden content package** | `.zip` downloaded for catch-up |
| 8 | Write down the **bundle symbolic name**, the **page resource type** and the **Core Components version** | used in Labs 9 and 11 |

## A5. Common issues and how the course handles them {: .flow}

| Issue | How it is handled |
|---|---|
| The generated project must target the LTS API jar | Lab 1 patch; the symptom of skipping it is shown |
| Embedded Core Components can clash with the platform's | Lab 1, step 4 |
| First build is slow or blocked by a proxy | Warm cache (A3) |
| A learner falls behind | `apply-module.sh` catch-up; rule: *nobody is stuck for more than 5 minutes* |
| Service-user mapping uses the wrong bundle name | Lab 1 records the real name; Lab 11 uses it |
| A console path differs on your build | `preflight.sh` flags it; adjust Appendix A |

## A6. Agenda: two days, 330 minutes of lab time each {: .flow}

**Day 1: build the site and its first component (09:00 to 15:45)**

| Time | Min | Lab | Topic |
|---|---|---|---|
| 09:00 | 30 | L0 | Orientation, consoles, pre-flight |
| 09:30 | 50 | L1 | Generate, patch for LTS, build, deploy |
| 10:20 | 15 | | *Break* |
| 10:35 | 30 | L2 | Repository and Sling orientation |
| 11:05 | 40 | L3 | Editable template, policies, pages |
| 11:45 | 40 | L4 | Header and footer components |
| 12:25 | 45 | | *Lunch* |
| 13:10 | 45 | L5 | Course Card: dialog and HTL |
| 13:55 | 45 | L6 | Sling Model and unit tests |
| 14:40 | 15 | | *Break* |
| 14:55 | 50 | L7 | Client libraries and the Style System |

**Day 2: services, data and automation (09:00 to 15:45)**

| Time | Min | Lab | Topic |
|---|---|---|---|
| 09:00 | 35 | L8 | OSGi service and configuration |
| 09:35 | 35 | L9 | Servlet |
| 10:10 | 25 | L10 | DAM image field |
| 10:35 | 15 | | *Break* |
| 10:50 | 45 | L11 | Scheduler, service user, logging |
| 11:35 | 55 | L12 | Workflow |
| 12:30 | 45 | | *Lunch* |
| 13:15 | 40 | L13 | Testing and the debugging toolbox |
| 13:55 | 25 | L14 | Packages, replication, Dispatcher, security review |
| 14:20 | 15 | | *Break* |
| 14:35 | 70 | L15 | Assessment: build, demo, quiz, retro |

**If a session runs behind, shorten in this order:** Lab 14's Dispatcher discussion (10 min), Lab 12's stretch (15 min), Lab 7's CSS exploration (10 min), Lab 10 (25 min, the image is optional). **Never cut:** Labs 1, 3, 5, 6, 9 and 11.



### Release and environment validation status

This workshop repository contains **sample source and smoke-test scripts**, not a pre-generated Maven project. It has been statically reviewed and corrected for cross-lab dependencies. A complete clean-instance run against **AEM 6.5 LTS SP2** (build, install, authoring UI, workflow, and publish/Dispatcher) is still required before claiming end-to-end certification. See `docs/INSTRUCTOR-VALIDATION.md`.


---

# Part B: Day 1, Build the Site

## Lab 0. Orientation, consoles and pre-flight (30 min)

**Objective:** everyone can reach the ten consoles a developer uses daily, and every machine passes pre-flight.

**Mode:** guided scavenger hunt in pairs.

### Whiteboard (5 min)

```
 Visitor -> CDN -> Dispatcher (cache + filter) -> PUBLISH :4503 <- replication <- AUTHOR :4502
                                                                        |
        Sling   maps URLs to resources and resources to scripts/servlets
        OSGi    your Java: bundles, components (DS), configuration
        Oak/JCR the repository: /apps  /libs  /conf  /content  /content/dam  /var  /home
        Java 17/21 on AEM 6.5 LTS (internally "AEM 6.6")
```

Four ideas: **everything is content** · **/libs is Adobe's, /apps is yours** · **author is not publish** · **LTS = the 6.5 experience on a modern foundation** (Java 17/21, `javax.*`, API jar `6.6.0` + `apis`).

### Pre-flight (10 min)

```bash
export KIT=~/trainingportal-lts-lab-kit          # where you unzipped the companion kit
export PATH="$KIT/scripts:$PATH"                 # makes preflight.sh, verify.sh, apply-module.sh available
export AEM_USER=admin
export AEM_PASS='<your local dev password>'     # local only; never commit it
preflight.sh
```

It checks Java, Maven, the instance, which **service pack** is installed, which **Core Components** are already on the instance, and every console URL below. Fix any FAIL before continuing.

### Scavenger hunt (15 min)

| # | Find | URL on author |
|---|---|---|
| 1 | Version and service pack | `/system/console/status-productinfo` |
| 2 | Sites console | `/sites.html/content` |
| 3 | Assets console | `/assets.html/content/dam` |
| 4 | Templates | `/libs/wcm/core/content/sites/templates.html/conf` |
| 5 | Content Fragment Models | `/libs/dam/cfm/models/console/content/models.html/conf` |
| 6 | Workflow Models | `/libs/cq/workflow/admin/console/content/models.html` |
| 7 | CRXDE Lite | `/crx/de/index.jsp` |
| 8 | Package Manager | `/crx/packmgr/index.jsp` |
| 9 | OSGi bundles / configuration / components | `/system/console/bundles`, `/configMgr`, `/components` |
| 10 | Log support | `/system/console/slinglog` |

**Debrief:** why must CRXDE and `/system/console` never be reachable on publish? Where do you look when a bundle says *Installed* instead of *Active*?

---

## Lab 1. Generate, patch for LTS, build and deploy (50 min)

**Objective:** a project generated from the Adobe archetype, adapted to the LTS API jar, deployed, with a verified **Active** bundle.

> **Rule:** the installed service pack decides what is correct. The archetype's `aemVersion` is only a compatibility selector, never a service-pack statement.

### Step 1: confirm the environment (5 min)

```bash
java -version          # expect 21
mvn -version
git --version
```
Open `/system/console/status-productinfo` and note the service pack.

### Step 2: generate the project (5 min)

```bash
mkdir -p ~/aem-workspace && cd ~/aem-workspace

mvn -B org.apache.maven.plugins:maven-archetype-plugin:3.3.1:generate \
  -DarchetypeGroupId=com.adobe.aem \
  -DarchetypeArtifactId=aem-project-archetype \
  -DarchetypeVersion=57 \
  -DaemVersion=6.5.17 \
  -DappTitle="Training Portal" \
  -DappId=trainingportal \
  -DgroupId=com.trainingportal \
  -Dversion=1.0.0-SNAPSHOT \
  -DfrontendModule=general \
  -DincludeExamples=n \
  -DincludeDispatcherConfig=n

cd trainingportal
git init && git add -A && git commit -m "Generated project (archetype 57)"
```

**Notes:** do not pass `-Dpackage` (the archetype appends `.core`, so it would create `com.trainingportal.core.core`); `artifactId` defaults to `appId`; `-DincludeDispatcherConfig=n` avoids symlink problems on Windows.

### Step 3: patch the API jar for LTS (5 min)

```bash
patch-archetype-for-lts.py . --check    # preview
patch-archetype-for-lts.py .             # apply
git diff --stat && git commit -am "Target AEM 6.5 LTS API jar (uber-jar 6.6.0, apis)"
```

It sets every `com.adobe.aem:uber-jar` dependency (or the property feeding it) to **6.6.0** and adds `<classifier>apis</classifier>`.

> **Symptom if you skip it:** hundreds of "package … does not exist" errors, or `ZipException: zip END header not found`. The plain `uber-jar-6.6.0.jar` is a 2 KB stub; the real API is in the `apis` classifier jar.

### Step 4: Core Components: do not embed an older copy (10 min)

LTS service packs ship their own Core Components. The archetype's `all` package embeds one for older 6.5 service packs.

1. Read the pre-flight section "Core Components already on the instance" (or search `core.wcm.components` in Bundles and in Package Manager).
2. Find the references: `grep -rn "core.wcm.components" --include=pom.xml .`
3. **Recommended:** in `all/pom.xml`, comment out the `<embedded>` entry for `core.wcm.components.all` and its matching `<dependency>`. Keep the `provided` compile-time dependencies in `core` and `ui.apps`.
4. **Alternative:** keep the embed only if its version is **equal or newer** than the installed one.

> **Note:** the exact element layout in `all/pom.xml` depends on the archetype release. If removing the embed breaks the build with a package-dependency error, restore it and set the `core.wcm.components.version` property in the root `pom.xml` to the version the instance reports.

Also read, but do **not** change, the compiler release in the generated POMs. Java 11 bytecode runs on 21; raising it needs a matching `bnd` plugin version.

### Step 5: build and deploy (15 min)

```bash
mvnd() { mvn "$@" -Dvault.user="$AEM_USER" -Dvault.password="$AEM_PASS" \
                  -Dsling.user="$AEM_USER" -Dsling.password="$AEM_PASS"; }

mvnd clean install -PautoInstallSinglePackage
```

(Credentials on the command line are visible in `ps` on a shared machine. That is acceptable on a personal dev box; otherwise use a permission-`600` Maven profile.) One command builds, tests and deploys. The first run takes several minutes because `ui.frontend` downloads Node and npm packages.

**Fast-deploy table for the rest of the course:**

| You changed | Run (from the project root) | Time |
|---|---|---|
| Java in `core` | `(cd core && mvnd clean install -PautoInstallBundle)` | ~15 s |
| Components or clientlibs in `ui.apps` | `(cd ui.apps && mvnd clean install -PautoInstallPackage)` | ~10 s |
| OSGi config in `ui.config` | `(cd ui.config && mvnd clean install -PautoInstallPackage)` | ~10 s |
| Everything | `mvnd clean install -PautoInstallSinglePackage` | minutes |
| Tests only | `(cd core && mvn clean test)` | ~10 s |

### Step 6: verify (5 min)

```bash
verify.sh l01
```

1. `/crx/packmgr/index.jsp` lists the `trainingportal.all` package as installed.
2. `/system/console/bundles`: the core bundle is **Active**. **Write down its Symbolic Name** (expected `com.trainingportal.core`). Lab 11 needs it.
3. CRXDE Lite: `/apps/trainingportal` exists.
4. Open `/editor.html/content/trainingportal/us/en.html`.

### If it fails

| Symptom | Cause and fix |
|---|---|
| `Could not resolve com.adobe.aem:uber-jar:6.6.0` | Adobe repository profile missing in `settings.xml` |
| Hundreds of "package does not exist" | `apis` classifier missing: rerun the patch |
| Bundle **Installed** or **Resolved** | Bundles, then the bundle, then *Imported packages*: a red entry is the unresolved package. Usually `jakarta.*` or Guava. Read `error.log`. |
| Package install error mentioning core components | Step 4 (embed clash) |
| `frontend-maven-plugin` cannot download Node | Proxy or offline: use the warm cache or set proxy options |

---

## Lab 2. Repository and Sling orientation (30 min)

**Objective:** predict, for any URL, which resource and which script handle it, and know your project's page resource type.

### Module map (5 min)

| Module | Holds | Deployed as |
|---|---|---|
| `core` | Java: models, services, servlets, schedulers, workflow steps | OSGi bundle |
| `ui.apps` | `/apps/trainingportal`: components (HTL, dialogs), clientlibs | content package (code) |
| `ui.config` | `/apps/trainingportal/osgiconfig`: OSGi settings by run mode | content package |
| `ui.content` | `/content`, `/conf` starter content, template types, policies | content package (mutable) |
| `ui.frontend` | webpack build that outputs a clientlib into `ui.apps` | feeds `ui.apps` |
| `all` | one package embedding the others | **the one you deploy** |

`/apps` application · `/content` pages · `/conf` editable templates and policies · `/content/dam` assets · `/libs` **product-owned, never edit**.

### Exercise 1: URL anatomy (10 min)

```
/content/trainingportal/us/en.tidy.2.json/suffix?x=1
 resource path                |  selectors  | ext | suffix | query
```
Try in the browser and note what changes:

| URL | Learn |
|---|---|
| `/content/trainingportal/us/en.json` | default GET servlet renders a node as JSON |
| `/content/trainingportal/us/en.1.json` | selector `1` = depth |
| `/content/trainingportal/us/en.tidy.2.json` | `tidy` pretty-prints |
| `/content/trainingportal/us/en/jcr:content.json` | the content node of the page |

Do **not** use `.infinity.json` on a large tree.

### Exercise 2: find the rendering component (10 min)

1. CRXDE: open `/content/trainingportal/us/en/jcr:content`. Read `sling:resourceType` and `cq:template`. **Write down the page resource type** (expected `trainingportal/components/page`). Lab 9 needs it.
2. Note that the `cq:Page` node `/content/trainingportal/us/en` has **no** `sling:resourceType`; its type is `cq:Page`. Only `jcr:content` carries the component type. *This is why Lab 9's URL contains `/jcr:content`.*
3. Open `/system/console/jcrresolver`, resolve `/content/trainingportal/us/en.html`.
4. Open `/system/console/servletresolver`, same path, method `GET`: read the ordered candidate scripts.
5. Load the page, then open `/system/console/requests` and click the latest request.

### Exercise 3: your first look at a package (5 min)

`/crx/packmgr/index.jsp`: click `trainingportal.ui.apps`, open **Contents** and **Filter**. This is exactly what Maven built from `ui.apps`.

### Checkpoint

- Which part of `/a/b.sel.html/x` is the selector? (`sel`)
- Why is a JSON rendering of a node a poor public API? (It exposes repository structure.)

---

## Lab 3. Editable template, policies and pages (40 min)

**Objective:** an enabled **Course Page** template and the pages `home`, `courses` and `about`.

> If the archetype created a usable template you may reuse it; this lab creates its own so everyone sees every part.

### Talk (8 min)

An editable template (`/conf/trainingportal/settings/wcm/templates/<name>`) has three parts:

| Part | Purpose |
|---|---|
| `structure` | locked on every page (header, footer, the container) |
| `initial` | starting content copied into each new page; authors may change it |
| `policies` | reusable rules per component (allowed components, styles, defaults) |

Policies are shared: edit one and every template using it changes. Allowing a **component group** also allows any component you later add to that group.

### Steps (32 min)

1. *Tools → General → Templates →* **trainingportal** *→ Create*. Choose the generic **page template type**, title **Course Page**. Open it.
2. In **Structure**, select the **Layout Container → Policy**. Create a policy **Course Container**. Under **Allowed Components** tick **Training Portal - Content** and **Training Portal - Structure** (the group names come from `appTitle`). Save.
3. Switch to **Initial Content** and add a **Title**.
4. **Page Information → Enable**.
5. In *Sites*, under `trainingportal / us / en`, **Create → Page** with the **Course Page** template: `home`, `courses`, `about` (Title may be capitalised; the *name* is lower-case).
6. Open each in the editor and type something in the Title.

### Verify
```bash
verify.sh l03
```
CRXDE: `/conf/trainingportal/settings/wcm/templates/course-page/{structure,initial,policies}` exist, and each new page has `jcr:content/cq:template` pointing there.

**Discussion:** adding a component to *structure* reaches existing pages immediately; adding to *initial content* affects new pages only. Why?

---

## Lab 4. Header and footer (40 min)

**Objective:** a Header and Footer placed once in the template structure and shown on every page.

> **Teaching vs production.** Production sites normally build header and footer as **Experience Fragments** referenced from the template, with Core **Navigation**. These components exist to teach dialogs, HTL and a request-adaptable model.

### What is in the module

`SiteNavModel` builds the menu from the siblings of the **Home page** you configure, so the menu follows the content tree instead of being hard-coded.

```bash
apply-module.sh l04 <project>
```


**Source: `modules/l04-header-footer/core/src/main/java/com/trainingportal/core/models/SiteNavModel.java`**

```java
package com.trainingportal.core.models;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
import java.util.List;

import javax.annotation.PostConstruct;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.resource.ResourceResolver;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ScriptVariable;
import org.apache.sling.models.annotations.injectorspecific.SlingObject;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

import com.day.cq.wcm.api.Page;
import com.day.cq.wcm.api.PageManager;

/**
 * Builds the header navigation from the siblings of the configured home page, so the menu follows
 * the content tree instead of being hard-coded. (For production sites prefer the Core Navigation
 * component; this model exists to teach PageManager and the request-adaptable model pattern.)
 */
@Model(adaptables = SlingHttpServletRequest.class, defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL)
public class SiteNavModel {

    static final String SITE_PREFIX = "/content/trainingportal/";
    static final int MAX_ITEMS = 10;

    /** One menu entry. Public and immutable; HTL reads it through its getters. */
    public static final class NavItem {
        private final String title;
        private final String url;
        private final boolean active;

        NavItem(String title, String url, boolean active) {
            this.title = title;
            this.url = url;
            this.active = active;
        }

        public String getTitle() {
            return title;
        }

        public String getUrl() {
            return url;
        }

        public boolean isActive() {
            return active;
        }
    }

    @SlingObject
    private ResourceResolver resolver;

    @ScriptVariable
    private Page currentPage;

    @ValueMapValue
    private String homeLink;

    private List<NavItem> items = Collections.emptyList();

    @PostConstruct
    protected void init() {
        if (!isAllowedHome(homeLink)) {
            return;
        }
        PageManager pageManager = resolver.adaptTo(PageManager.class);
        Page home = pageManager == null ? null : pageManager.getPage(homeLink);
        if (home == null || home.getParent() == null) {
            return;
        }
        List<NavItem> result = new ArrayList<>();
        Iterator<Page> children = home.getParent().listChildren();
        while (children.hasNext() && result.size() < MAX_ITEMS) {
            Page child = children.next();
            if (child.isHideInNav()) {
                continue;
            }
            boolean active = currentPage != null && currentPage.getPath().equals(child.getPath());
            result.add(new NavItem(titleOf(child), child.getPath() + ".html", active));
        }
        items = Collections.unmodifiableList(result);
    }

    public List<NavItem> getItems() {
        return items;
    }

    static boolean isAllowedHome(String path) {
        return path != null && path.startsWith(SITE_PREFIX) && !path.contains("..");
    }

    private static String titleOf(Page page) {
        if (page.getNavigationTitle() != null) {
            return page.getNavigationTitle();
        }
        return page.getTitle() != null ? page.getTitle() : page.getName();
    }
}
```

Points to explain: `@ScriptVariable` gives the model the page being rendered; `homeLink` is validated to stay inside the site; `NavItem` is a small immutable class that HTL reads through getters.


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/header/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="TP Header"
    jcr:description="Site header with logo text and navigation (training example)"
    componentGroup="Training Portal - Structure"/>
```


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/header/_cq_dialog/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="TP Header"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <logoText
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                        fieldLabel="Logo text"
                        maxlength="60"
                        name="./logoText"
                        required="{Boolean}true"/>
                    <homeLink
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                        fieldDescription="The home page. The menu lists this page and its sibling pages."
                        fieldLabel="Home page"
                        name="./homeLink"
                        required="{Boolean}true"
                        rootPath="/content/trainingportal"/>
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/header/header.html`**

```html
<sly data-sly-use.nav="com.trainingportal.core.models.SiteNavModel"
     data-sly-use.template="core/wcm/components/commons/v1/templates.html"/>
<header data-sly-test="${properties.logoText}" class="tp-header">
    <a class="tp-header__logo" href="${properties.homeLink @ extension='html'}">${properties.logoText}</a>
    <nav data-sly-test="${nav.items}" class="tp-header__nav" aria-label="Main">
        <a data-sly-list.item="${nav.items}"
           class="tp-header__link${item.active ? ' is-active'}"
           href="${item.url}"
           aria-current="${item.active ? 'page' : false}">${item.title}</a>
    </nav>
</header>
<sly data-sly-call="${template.placeholder @ isEmpty=!properties.logoText}"/>
```


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/footer/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="TP Footer"
    jcr:description="Site footer (training example)"
    componentGroup="Training Portal - Structure"/>
```


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/footer/_cq_dialog/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="TP Footer"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <footerText
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                        fieldLabel="Footer text"
                        maxlength="120"
                        name="./footerText"
                        required="{Boolean}true"/>
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```


**Source: `modules/l04-header-footer/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/footer/footer.html`**

```html
<sly data-sly-use.template="core/wcm/components/commons/v1/templates.html"/>
<footer data-sly-test="${properties.footerText}" class="tp-footer">
    <p class="tp-footer__text">${properties.footerText}</p>
</footer>
<sly data-sly-call="${template.placeholder @ isEmpty=!properties.footerText}"/>
```

### Steps

1. Deploy: `(cd core && mvnd clean install -PautoInstallBundle)` then `(cd ui.apps && mvnd clean install -PautoInstallPackage)`.
2. Template editor, **Structure** mode. If your template already contains header/footer Experience Fragments, delete them from this template's structure to avoid duplicates.
3. Make sure the **Layout Container** policy allows the **Training Portal - Structure** group (Lab 3).
4. Drag **TP Header** above the main container and **TP Footer** below it. Open their dialogs and set:

   | Field | Value |
   |---|---|
   | Logo text | `Training Portal` |
   | Home page | `/content/trainingportal/us/en/home` |
   | Footer text | `Training Portal` |

5. Open *home*, *courses*, *about*: all show the header (with three menu entries) and footer.

### Verify
```bash
verify.sh l04
```
**Break it:** set Home page to `/content/other/x`. The menu disappears and no error is shown to visitors; read `SiteNavModel.isAllowedHome` to see why.

---

## Lab 5. Course Card: dialog and HTL (45 min)

**Objective:** a complete component (definition, authoring dialog, HTL) used twice on the Courses page.

### Talk (8 min)

A component is a folder with a `cq:Component` definition, an optional `_cq_dialog` (stored as `cq:dialog`) and an HTL script named after it. HTL **escapes every expression for its context** (`${properties.title}` is HTML-escaped, `href` is URI-checked). Opting out with `@ context='unsafe'` should be a code-review red flag.

Dialog field names: `./title`, `./description`, `./duration`, `./courseLink`. `duration` is **minutes**.

### Steps

```bash
apply-module.sh l05 <project>
(cd ui.apps && mvnd clean install -PautoInstallPackage)
```


**Source: `modules/l05-course-card-component/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="cq:Component"
    jcr:title="Course Card"
    jcr:description="A course teaser card (training example)"
    componentGroup="Training Portal - Content"/>
```


**Source: `modules/l05-course-card-component/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/_cq_dialog/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="Course Card"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <title
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                        fieldLabel="Title"
                        maxlength="80"
                        name="./title"
                        required="{Boolean}true"/>
                    <description
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                        fieldLabel="Description"
                        maxlength="300"
                        name="./description"/>
                    <duration
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/numberfield"
                        fieldDescription="Length in minutes"
                        fieldLabel="Duration (minutes)"
                        min="1"
                        name="./duration"
                        step="1"/>
                    <courseLink
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                        fieldDescription="A page inside /content/trainingportal"
                        fieldLabel="Course page"
                        name="./courseLink"
                        rootPath="/content/trainingportal"/>
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```


**Source: `modules/l05-course-card-component/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/coursecard.html`**

```html
<!--/*
  Version 1 (Lab 5): reads authored properties directly. HTL escapes every ${...} for its context.
  Lab 6 moves the logic into a Sling Model; Lab 10 adds the image.
*/-->
<sly data-sly-use.template="core/wcm/components/commons/v1/templates.html"/>
<article data-sly-test="${properties.title}" class="tp-course-card">
    <div class="tp-course-card__body">
        <h3 class="tp-course-card__title">${properties.title}</h3>
        <p data-sly-test="${properties.duration}" class="tp-course-card__meta">${properties.duration} minutes</p>
        <p data-sly-test="${properties.description}">${properties.description}</p>
    </div>
</article>
<sly data-sly-call="${template.placeholder @ isEmpty=!properties.title}"/>
```

1. Open **Courses**, add a **Course Card** (it appears under *Training Portal - Content* because the group is allowed).
2. Card 1: **AEM Fundamentals** / **Learn the basics** / **45**. Card 2: **Component Development** / **Build your own components** / **120**.
3. Find a card in CRXDE. **It is stored on the component node**, for example `/content/trainingportal/us/en/courses/jcr:content/root/container/container/coursecard`, **not** on the page's `jcr:content`. Read its properties.

### Two experiments

1. **Escaping:** set a title to `<script>alert(1)</script>`; view source: it is escaped by HTL.
2. **Required field:** clear the title in the dialog; Save is blocked (`required`).

### Verify
```bash
verify.sh l05
```

---

## Lab 6. Sling Model and unit tests (45 min)

**Objective:** move the logic into a validated, unit-tested Java class: level, labels, link.

### Talk (10 min)

| Decision | Choice | Why |
|---|---|---|
| Adaptable | `SlingHttpServletRequest` | request resolver, current page, policies |
| Strategy | `OPTIONAL` | a missing property gives `null`, not a failed model |
| Injectors | `@ValueMapValue` now; `@OSGiService` in Lab 8 | specific injectors; avoid the generic `@Inject` |
| `resourceType` | set | enables `.model.json` through the exporter |
| Validation | in `@PostConstruct` | never trust the repository |

> **LTS rule:** use `javax.annotation.PostConstruct`. A `jakarta.annotation.PostConstruct` import compiles in some setups but is **silently ignored** at runtime by the LTS Sling Models implementation.

Business rule: **Beginner when duration is 60 minutes or less, Intermediate above.**

### Steps

```bash
apply-module.sh l06 <project>
```


**Source: `modules/l06-sling-model/core/src/main/java/com/trainingportal/core/models/CourseCardModel.java`**

```java
package com.trainingportal.core.models;

import javax.annotation.PostConstruct;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Exporter;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

/**
 * Sling Model behind the Course Card component.
 *
 * <ul>
 *   <li>{@code adaptables = SlingHttpServletRequest}: usable from HTL with {@code data-sly-use}.</li>
 *   <li>{@code resourceType}: lets the Jackson exporter serve {@code .model.json} for the component.</li>
 *   <li>Everything an author can type is validated here, never trusted.</li>
 * </ul>
 */
@Model(
    adaptables = SlingHttpServletRequest.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL,
    resourceType = CourseCardModel.RESOURCE_TYPE)
@Exporter(name = "jackson", extensions = "json")
public class CourseCardModel {

    public static final String RESOURCE_TYPE = "trainingportal/components/coursecard";

    /** Beginner up to and including this many minutes, Intermediate above it. */
    static final int BEGINNER_MAX_MINUTES = 60;

    private static final String SITE_PREFIX = "/content/trainingportal/";
    private static final String DAM_PREFIX = "/content/dam/trainingportal/";

    @ValueMapValue
    private String title;

    @ValueMapValue
    private String description;

    @ValueMapValue
    private Integer duration;

    @ValueMapValue
    private String courseLink;

    private String level;

    @PostConstruct
    protected void init() {
        if (duration == null || duration <= 0) {
            level = null;
        } else {
            level = duration <= BEGINNER_MAX_MINUTES ? "Beginner" : "Intermediate";
        }
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }

    public Integer getDuration() {
        return duration;
    }

    /** "45 minutes", or null when no valid duration was authored. */
    public String getDurationLabel() {
        return level == null ? null : duration + " minutes";
    }

    /** "Beginner" (up to 60 minutes) or "Intermediate"; null without a valid duration. */
    public String getLevel() {
        return level;
    }

    /** Lower-case form of the level, safe as a CSS modifier. */
    public String getLevelKey() {
        return level == null ? null : level.toLowerCase(java.util.Locale.ROOT);
    }

    /** Link to a page inside the site (".html" added when missing); null for anything else. */
    public String getLinkUrl() {
        if (courseLink == null || !courseLink.startsWith(SITE_PREFIX) || courseLink.contains("..")) {
            return null;
        }
        String lastSegment = courseLink.substring(courseLink.lastIndexOf('/') + 1);
        return lastSegment.contains(".") ? courseLink : courseLink + ".html";
    }

    public boolean isConfigured() {
        return title != null && !title.trim().isEmpty();
    }

    static boolean isDamPath(String path) {
        return path != null && path.startsWith(DAM_PREFIX) && !path.contains("..");
    }
}
```


**Source: `modules/l06-sling-model/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/coursecard.html`**

```html
<!--/*
  Version 2 (Lab 6): the Sling Model owns the logic (level, label, link validation).
  The template only renders. Compare with version 1.
*/-->
<sly data-sly-use.card="com.trainingportal.core.models.CourseCardModel"
     data-sly-use.template="core/wcm/components/commons/v1/templates.html"/>
<article data-sly-test="${card.configured}" class="tp-course-card">
    <div class="tp-course-card__body">
        <h3 class="tp-course-card__title">${card.title}</h3>
        <p class="tp-course-card__meta">
            <span data-sly-test="${card.level}"
                  class="tp-course-card__level tp-course-card__level--${card.levelKey @ context='styleToken'}">${card.level}</span>
            <span data-sly-test="${card.durationLabel}">${card.durationLabel}</span>
        </p>
        <p data-sly-test="${card.description}" class="tp-course-card__description">${card.description}</p>
        <a data-sly-test="${card.linkUrl}" class="tp-course-card__link" href="${card.linkUrl}">View course</a>
    </div>
</article>
<sly data-sly-call="${template.placeholder @ isEmpty=!card.configured}"/>
```


**Source: `modules/l06-sling-model/core/src/test/java/com/trainingportal/core/models/CourseCardModelTest.java`**

```java
package com.trainingportal.core.models;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertNull;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;

import io.wcm.testing.mock.aem.junit5.AemContext;
import io.wcm.testing.mock.aem.junit5.AemContextExtension;

@ExtendWith(AemContextExtension.class)
class CourseCardModelTest {

    private final AemContext context = new AemContext();

    @BeforeEach
    void setUp() {
        context.addModelsForClasses(CourseCardModel.class);
    }

    private CourseCardModel modelFor(Object... properties) {
        context.create().resource("/content/card", properties);
        context.currentResource("/content/card");
        CourseCardModel model = context.request().adaptTo(CourseCardModel.class);
        assertNotNull(model, "model should adapt from the request");
        return model;
    }

    @Test
    void fortyFiveMinutesIsBeginner() {
        CourseCardModel m = modelFor("title", "AEM Fundamentals", "duration", 45);
        assertTrue(m.isConfigured());
        assertEquals("Beginner", m.getLevel());
        assertEquals("45 minutes", m.getDurationLabel());
    }

    @Test
    void oneTwentyMinutesIsIntermediate() {
        assertEquals("Intermediate", modelFor("title", "x", "duration", 120).getLevel());
    }

    @Test
    void sixtyMinutesIsStillBeginner() {
        assertEquals("Beginner", modelFor("title", "x", "duration", 60).getLevel());
    }

    @Test
    void sixtyOneMinutesIsIntermediate() {
        assertEquals("Intermediate", modelFor("title", "x", "duration", 61).getLevel());
    }

    @Test
    void nullOrInvalidDurationHasNoLevel() {
        assertNull(modelFor("title", "x").getLevel());
    }

    @Test
    void zeroDurationHasNoLevel() {
        assertNull(modelFor("title", "x", "duration", 0).getLevel());
    }

    @Test
    void linkMustStayInsideTheSite() {
        assertEquals("/content/trainingportal/us/en/courses.html",
            modelFor("title", "x", "courseLink", "/content/trainingportal/us/en/courses").getLinkUrl());
    }

    @Test
    void foreignOrTraversalLinksAreRejected() {
        assertNull(modelFor("title", "x", "courseLink", "javascript:alert(1)").getLinkUrl());
    }

    @Test
    void traversalLinkIsRejected() {
        assertNull(modelFor("title", "x", "courseLink", "/content/trainingportal/../secret").getLinkUrl());
    }

    @Test
    void blankTitleIsNotConfigured() {
        assertFalse(modelFor("title", "   ").isConfigured());
    }
}
```

1. Run the tests: `(cd core && mvn clean test)`. The test class covers null duration, 45, 120 and the 60/61 boundary, the link rules and a blank title.
2. **Break it on purpose:** change `<=` to `<` in the level rule and watch `sixtyMinutesIsStillBeginner` fail. Restore it.
3. Deploy both: `core` (bundle) and `ui.apps`.
4. Reload Courses: each card shows a **Beginner** or **Intermediate** badge and "45 minutes" / "120 minutes".
5. Add `.model.json` to the card's repository path: you get the model as JSON.
6. Open `/system/console/status-slingmodels` and find `CourseCardModel` bound to `trainingportal/components/coursecard`.

### Explain
`@Model`, the adaptables, `@ValueMapValue`, `@PostConstruct`, and where `@OSGiService` comes in (Lab 8, extension).

### Verify
```bash
verify.sh l06
```

---

## Lab 7. Client libraries and the Style System (50 min)

**Objective:** a client library served through the publish-safe proxy path, included once per page, plus two author-selectable styles.

### Talk (8 min)

A client library is a `cq:ClientLibraryFolder` with `categories`. The properties that matter:

| Property | Meaning |
|---|---|
| `categories` | the names you include by (`trainingportal.training`) |
| `allowProxy=true` | serve through `/etc.clientlibs/...`. **Required** for anything under `/apps`, which must not be reachable on publish |
| `embed` / `dependencies` | bundle or order other categories |

`css.txt` and `js.txt` list the files (first line `#base=css` or `#base=js`). `?debugClientLibs=true` shows unminified files.

### Steps

```bash
apply-module.sh l07 <project>
```


**Source: `modules/l07-clientlibs/ui.apps/src/main/content/jcr_root/apps/trainingportal/clientlibs/clientlib-training/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0"
    jcr:primaryType="cq:ClientLibraryFolder"
    allowProxy="{Boolean}true"
    categories="[trainingportal.training]"/>
```


**Source: `modules/l07-clientlibs/ui.apps/src/main/content/jcr_root/apps/trainingportal/clientlibs/clientlib-training/css.txt`**

```txt
#base=css
training.css
```


**Source: `modules/l07-clientlibs/ui.apps/src/main/content/jcr_root/apps/trainingportal/clientlibs/clientlib-training/js.txt`**

```txt
#base=js
training.js
```


**Source: `modules/l07-clientlibs/ui.apps/src/main/content/jcr_root/apps/trainingportal/clientlibs/clientlib-training/css/training.css`**

```css
/* Training Portal styles (BEM). Style System classes land on the AEM wrapper <div>, so
   variants are written as ".cmp-featured .tp-course-card", not ".tp-course-card.cmp-featured". */

.tp-header {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 1rem 2rem;
    padding: .75rem 1.5rem;
    background: #1f2d3d;
    color: #fff;
}
.tp-header__logo {
    font-size: 1.25rem;
    font-weight: 700;
    color: #fff;
    text-decoration: none;
}
.tp-header__nav {
    display: flex;
    gap: 1rem;
}
.tp-header__link {
    color: #cfd8e3;
    text-decoration: none;
}
.tp-header__link:hover,
.tp-header__link.is-active {
    color: #fff;
    text-decoration: underline;
}
.tp-footer {
    padding: 1rem 1.5rem;
    background: #f1f3f5;
    color: #495057;
    text-align: center;
}

.tp-course-card {
    display: flex;
    flex-direction: column;
    margin-bottom: 1rem;
    border: 1px solid #d5d5d5;
    border-radius: 8px;
    overflow: hidden;
    background: #fff;
}
.tp-course-card__image {
    width: 100%;
    max-height: 200px;
    object-fit: cover;
}
.tp-course-card__body {
    padding: 1rem;
}
.tp-course-card__title {
    margin: 0 0 .5rem;
}
.tp-course-card__meta {
    display: flex;
    gap: .75rem;
    align-items: center;
    margin: 0 0 .5rem;
    color: #505050;
}
.tp-course-card__level {
    padding: .1rem .6rem;
    border-radius: 999px;
    font-size: .8rem;
    font-weight: 600;
    background: #e6f4ea;
    color: #1e6b3a;
}
.tp-course-card__level--intermediate {
    background: #fff4d6;
    color: #7a5a00;
}
.tp-course-card__description {
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
    overflow: hidden;
}
.tp-course-card.is-expanded .tp-course-card__description {
    display: block;
}

/* Style System variants (Lab 7): Featured = cmp-featured, Compact = cmp-compact */
.cmp-featured .tp-course-card {
    border: 2px solid #1473e6;
    box-shadow: 0 4px 14px rgba(20, 115, 230, .25);
}
.cmp-featured .tp-course-card__title {
    font-size: 1.6rem;
}
.cmp-compact .tp-course-card__image,
.cmp-compact .tp-course-card__description {
    display: none;
}
.cmp-compact .tp-course-card__body {
    padding: .5rem 1rem;
}
```


**Source: `modules/l07-clientlibs/ui.apps/src/main/content/jcr_root/apps/trainingportal/clientlibs/clientlib-training/js/training.js`**

```js
/*
 * Course card: clicking the card title toggles the clamped description.
 * No inline script, no libraries (CSP friendly). Re-initialises when the page editor re-renders.
 */
(function () {
    "use strict";

    var CARD = ".tp-course-card";
    var TITLE = ".tp-course-card__title";

    function init(card) {
        if (card.getAttribute("data-tp-init") === "true") {
            return;
        }
        card.setAttribute("data-tp-init", "true");
        var title = card.querySelector(TITLE);
        if (!title) {
            return;
        }
        title.setAttribute("tabindex", "0");
        title.setAttribute("role", "button");
        title.setAttribute("aria-expanded", "false");
        function toggle() {
            var expanded = card.classList.toggle("is-expanded");
            title.setAttribute("aria-expanded", expanded ? "true" : "false");
        }
        title.addEventListener("click", toggle);
        title.addEventListener("keydown", function (event) {
            if (event.key === "Enter" || event.key === " ") {
                event.preventDefault();
                toggle();
            }
        });
    }

    function initAll() {
        var cards = document.querySelectorAll(CARD);
        for (var i = 0; i < cards.length; i++) {
            init(cards[i]);
        }
    }

    function start() {
        initAll();
        if (document.body && "MutationObserver" in window) {
            new MutationObserver(initAll).observe(document.body, { childList: true, subtree: true });
        }
    }

    if (document.readyState === "loading") {
        document.addEventListener("DOMContentLoaded", start);
    } else {
        start();
    }
})();
```

**Include them once, in the page component** (not in every card, or ten cards would load the library ten times). **Do not replace** the archetype's existing includes: open the files and **append**.

`ui.apps/…/components/page/customheaderlibs.html`, add at the end:

```html
<sly data-sly-use.tpLibs="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${tpLibs.css @ categories='trainingportal.training'}"/>
```

`ui.apps/…/components/page/customfooterlibs.html`, add at the end:

```html
<sly data-sly-use.tpLibs="/libs/granite/sightly/templates/clientlib.html"
     data-sly-call="${tpLibs.js @ categories='trainingportal.training'}"/>
```
CSS in the head avoids an unstyled flash; JS at the end of the body does not block rendering.

### Style System

1. In the **Course Page template editor**, switch to **Structure** and select the editable container that permits Course Cards. Open its **Policy** and confirm `Training Portal - Content` is allowed. Configure the **Course Card component policy** for this template/container (do not add a sample card to Initial Content merely to set its policy). If the policy action is unavailable, inspect the template policy mapping in `/conf/trainingportal/settings/wcm/policies` and ask the instructor to validate the SP2 UI sequence before proceeding.
2. **Styles** tab → **Add a style group** *Appearance* → add styles **Featured** = `cmp-featured` and **Compact** = `cmp-compact`. Save.
3. On the *Courses* page select an **existing** Course Card → paintbrush icon → choose **Featured**, then **Compact**. If neither appears, the policy is not applied to the component in the correct container; correct the template policy association and retry. Verify the style class on the wrapper in browser DevTools.

**Why the CSS looks the way it does:** AEM puts the style class on a **wrapper `<div>` around the component**, not on the `<article>`. So the CSS targets `.cmp-featured .tp-course-card` (a descendant). Inspect the DOM to see the wrapper.

### Verify
```bash
(cd ui.apps && mvnd clean install -PautoInstallPackage)
verify.sh l07
```

1. In the browser **Network** tab: `clientlib-training.min.css` and `.js` load from `/etc.clientlibs/trainingportal/clientlibs/…`.
2. `?debugClientLibs=true` shows the raw files.
3. **Break it:** remove `allowProxy`, redeploy: the CSS 404s. Restore. (The classic "works on author, broken on publish" bug.)



---

# Part C: Day 2, Services, Data and Automation

> **Morning ritual (5 min):** `verify.sh all`. Anyone with a FAIL for Labs 1 to 7 runs `apply-module.sh <lab> <project>` for that lab and redeploys. Do not start new work on a broken base.

## Lab 8. OSGi service and configuration (35 min)

**Objective:** a Declarative Services component with a typed configuration you change live and then commit to source control.

### Talk (8 min)

- Your Java lives in an **OSGi bundle**; "Installed but not Active" means an import could not be resolved.
- **Declarative Services** components are classes annotated `@Component`. A **metatype** (`@ObjectClassDefinition` + `@Designate`) describes their configuration, which then appears in `/system/console/configMgr`.
- Configuration ships from code as `<pid>.cfg.json` under `ui.config/.../osgiconfig/<runmode folder>/`. Run-mode folders: `config` (everywhere), `config.author`, `config.publish`, `config.dev`.
- Configuration is **input**: validate and clamp it.

### Steps

```bash
apply-module.sh l08 <project>
```


**Source: `modules/l08-osgi-service/core/src/main/java/com/trainingportal/core/services/CourseService.java`**

```java
package com.trainingportal.core.services;

import java.util.List;

/** Course-related business rules that depend on configuration. */
public interface CourseService {

    /** Configured maximum number of courses to show; always between 1 and 100. */
    int getMaximumCourses();

    /** Returns at most {@link #getMaximumCourses()} items (a view; the input is not modified). */
    <T> List<T> limit(List<T> courses);
}
```


**Source: `modules/l08-osgi-service/core/src/main/java/com/trainingportal/core/services/impl/CourseServiceImpl.java`**

```java
package com.trainingportal.core.services.impl;

import java.util.Collections;
import java.util.List;

import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Modified;
import org.osgi.service.metatype.annotations.AttributeDefinition;
import org.osgi.service.metatype.annotations.Designate;
import org.osgi.service.metatype.annotations.ObjectClassDefinition;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.trainingportal.core.services.CourseService;

/**
 * An OSGi Declarative Services component with a typed, metatype-described configuration.
 * The configuration can be edited at /system/console/configMgr and shipped from source control
 * as {@code com.trainingportal.core.services.impl.CourseServiceImpl.cfg.json}.
 */
@Component(service = CourseService.class)
@Designate(ocd = CourseServiceImpl.Config.class)
public class CourseServiceImpl implements CourseService {

    static final int MIN = 1;
    static final int MAX = 100;
    static final int DEFAULT = 10;

    private static final Logger LOG = LoggerFactory.getLogger(CourseServiceImpl.class);

    @ObjectClassDefinition(
        name = "Training Portal - Course Service",
        description = "Settings for the course listing rules")
    public @interface Config {

        @AttributeDefinition(name = "Maximum courses", description = "How many courses to show (1 to 100)",
            min = "1", max = "100")
        int maximumCourses() default DEFAULT;
    }

    private volatile int maximumCourses = DEFAULT;

    @Activate
    @Modified
    protected void configure(Config config) {
        maximumCourses = clamp(config.maximumCourses());
        LOG.info("CourseService configured: maximumCourses={}", maximumCourses);
    }

    @Override
    public int getMaximumCourses() {
        return maximumCourses;
    }

    @Override
    public <T> List<T> limit(List<T> courses) {
        if (courses == null) {
            return Collections.emptyList();
        }
        return courses.size() <= maximumCourses ? courses : courses.subList(0, maximumCourses);
    }

    /** Configuration is input too: out-of-range values are corrected, not trusted. */
    static int clamp(int value) {
        return Math.max(MIN, Math.min(value, MAX));
    }
}
```


**Source: `modules/l08-osgi-service/core/src/main/java/com/trainingportal/core/services/package-info.java`**

```java
@org.osgi.annotation.versioning.Version("1.0.0")
package com.trainingportal.core.services;
```

*Why `package-info.java`?* A versioned package is **exported** by the bundle (the archetype's `bnd` configuration exports versioned packages). Without it, other bundles and HTL scripts cannot load the types.


**Source: `modules/l08-osgi-service/ui.config/src/main/content/jcr_root/apps/trainingportal/osgiconfig/config/com.trainingportal.core.services.impl.CourseServiceImpl.cfg.json`**

```json
{
  "maximumCourses": 10
}
```


**Source: `modules/l08-osgi-service/core/src/test/java/com/trainingportal/core/services/impl/CourseServiceImplTest.java`**

```java
package com.trainingportal.core.services.impl;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;

import org.junit.jupiter.api.Test;

class CourseServiceImplTest {

    @Test
    void clampKeepsValuesInRange() {
        assertEquals(1, CourseServiceImpl.clamp(0));
        assertEquals(1, CourseServiceImpl.clamp(-50));
        assertEquals(5, CourseServiceImpl.clamp(5));
        assertEquals(100, CourseServiceImpl.clamp(100000));
    }

    @Test
    void limitCutsTheList() {
        CourseServiceImpl service = new CourseServiceImpl();
        List<Integer> all = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12);
        assertEquals(10, service.limit(all).size());   // default is 10
    }

    @Test
    void limitHandlesNullAndShortLists() {
        CourseServiceImpl service = new CourseServiceImpl();
        assertTrue(service.limit(null).isEmpty());
        assertEquals(Collections.singletonList("a"), service.limit(Collections.singletonList("a")));
    }
}
```

1. `(cd core && mvn clean test)`, then deploy `core` and `ui.config`.
2. `/system/console/components`: find `CourseServiceImpl` (**active**).
3. `/system/console/configMgr`: search **Course Service**. The default is **10**. Change it to **5** and **Save**. The `@Modified` method applies it **live**.
4. The log shows `CourseService configured: maximumCourses=5` (error.log until Lab 11 gives you a dedicated log).
5. **Make it permanent:** a value saved in the console lives in that instance only. Edit the `.cfg.json` in source control to `5`, redeploy `ui.config`. *This is the difference between configuring a server and configuring a product.*
6. Out-of-range input: try **1000** in the console. The service reports and uses 100.

### Verify
```bash
verify.sh l08
```

### Extension
Inject the service into the card model with `@OSGiService` (a typical use of `@OSGiService`):

```java
@OSGiService
private CourseService courseService;

public int getMaximumCourses() { return courseService.getMaximumCourses(); }
```

---

## Lab 9. Servlet (35 min)

**Objective:** a safe, read-only JSON endpoint that proves the service configuration.

### Talk (8 min)

| | Resource-type bound (use this) | Path bound (avoid) |
|---|---|---|
| Annotation | `@SlingServletResourceTypes(resourceTypes, selectors, extensions, methods)` | `@SlingServletPaths("/bin/…")` |
| Security | Respects the repository ACLs of the resource being requested | Bypasses them; you must code the checks |
| Dispatcher | normal content URL | separate `/bin` URL to allow or deny |

Rules for every servlet: narrow registration (one type, one selector, one extension, GET only) · extend `SlingSafeMethodsServlet` for read-only · treat all input as untrusted · write JSON with Jackson, never by string concatenation · clean status codes, no stack traces.

> **Common pitfall.** `home.courseinfo.json` resolves to the `cq:Page` node, whose type is `cq:Page`. The page **component** type belongs to its `jcr:content` node. So the URL is `/content/trainingportal/us/en/home/jcr:content.courseinfo.json` and the servlet binds to the page **component** type you wrote down in Lab 2.

### Steps

```bash
apply-module.sh l09 <project>
```


**Source: `modules/l09-servlet/core/src/main/java/com/trainingportal/core/servlets/CourseInfoServlet.java`**

```java
package com.trainingportal.core.servlets;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

import javax.servlet.Servlet;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.api.SlingHttpServletResponse;
import org.apache.sling.api.servlets.HttpConstants;
import org.apache.sling.api.servlets.SlingSafeMethodsServlet;
import org.apache.sling.servlets.annotations.SlingServletResourceTypes;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Reference;
import org.osgi.service.component.propertytypes.ServiceDescription;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.trainingportal.core.services.CourseService;

/**
 * GET {@code <page>/jcr:content.courseinfo.json}
 *
 * <p>Bound to the page <b>component's</b> resource type (that is the type of the page's
 * {@code jcr:content} node; the {@code cq:Page} node itself has the type {@code cq:Page}),
 * to one selector and one extension, GET only. Requires a logged-in user: this is an internal
 * diagnostic endpoint and must not be reachable anonymously.</p>
 */
@Component(service = Servlet.class)
@SlingServletResourceTypes(
    resourceTypes = "trainingportal/components/page",
    methods = HttpConstants.METHOD_GET,
    selectors = "courseinfo",
    extensions = "json")
@ServiceDescription("Training Portal - course settings as JSON")
public class CourseInfoServlet extends SlingSafeMethodsServlet {

    private static final long serialVersionUID = 1L;
    private static final ObjectMapper MAPPER = new ObjectMapper();

    @Reference
    private transient CourseService courseService;

    @Override
    protected void doGet(SlingHttpServletRequest request, SlingHttpServletResponse response) throws IOException {
        String userId = request.getResourceResolver().getUserID();
        if (userId == null || "anonymous".equals(userId)) {
            response.sendError(SlingHttpServletResponse.SC_FORBIDDEN);
            return;
        }

        ObjectNode json = MAPPER.createObjectNode();
        json.put("maximumCourses", courseService.getMaximumCourses());

        response.setContentType("application/json");
        response.setCharacterEncoding(StandardCharsets.UTF_8.name());
        response.setHeader("X-Content-Type-Options", "nosniff");
        response.setHeader("Cache-Control", "no-store");
        MAPPER.writeValue(response.getWriter(), json);
    }
}
```

1. If the resource type you recorded in Lab 2 is not `trainingportal/components/page`, edit it in the annotation.
2. Deploy `core`.
3. In a browser logged in to author, open  
   `http://localhost:4502/content/trainingportal/us/en/home/jcr:content.courseinfo.json`  
   Expected: `{"maximumCourses":5}`.
4. Prove that it is not public:

   ```bash
   curl -i "http://localhost:4502/content/trainingportal/us/en/home/jcr:content.courseinfo.json"
   ```
   No credentials: you get a redirect to login or a 401/403/404, never the JSON.
5. `/system/console/servletresolver`: resolve the same path with method `GET`; your servlet is the first candidate.
6. Change the value in configMgr to 7: the endpoint reports 7 immediately.

### Verify
```bash
EXPECT_MAX=5 verify.sh l09
```

### If you get a 404
Check, in order: (1) the URL contains `/jcr:content`; (2) the resource type in the annotation equals the page's `sling:resourceType`; (3) `/system/console/components` shows `CourseInfoServlet` **active** (an *unsatisfied reference* means the service is missing); (4) selector and extension spelling.

### Discussion
"Read-only servlet" does **not** make the node read-only: a `POST` to a resource is handled by the default Sling POST servlet and limited only by repository ACLs. That is why anonymous users must have no write access on publish, and why the Dispatcher blocks POST to content.

---

## Lab 10. DAM image field (25 min)

**Objective:** an image on the Course Card from the DAM, with safe handling of missing and foreign paths.

### Steps

1. *Assets → Files → Create → Folder*: `trainingportal`, then inside it `courses`. Upload a sample JPG into `/content/dam/trainingportal/courses`.
2. Apply the module (replaces the dialog, model, test and HTL with the image-aware versions):

   ```bash
   apply-module.sh l10 <project>
   ```


**Source: `modules/l10-dam-image/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/_cq_dialog/.content.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jcr:root xmlns:sling="http://sling.apache.org/jcr/sling/1.0" xmlns:cq="http://www.day.com/jcr/cq/1.0" xmlns:jcr="http://www.jcp.org/jcr/1.0" xmlns:nt="http://www.jcp.org/jcr/nt/1.0"
    jcr:primaryType="nt:unstructured"
    jcr:title="Course Card"
    sling:resourceType="cq/gui/components/authoring/dialog">
    <content
        jcr:primaryType="nt:unstructured"
        sling:resourceType="granite/ui/components/coral/foundation/container">
        <items jcr:primaryType="nt:unstructured">
            <column
                jcr:primaryType="nt:unstructured"
                sling:resourceType="granite/ui/components/coral/foundation/container">
                <items jcr:primaryType="nt:unstructured">
                    <title
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textfield"
                        fieldLabel="Title"
                        maxlength="80"
                        name="./title"
                        required="{Boolean}true"/>
                    <description
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/textarea"
                        fieldLabel="Description"
                        maxlength="300"
                        name="./description"/>
                    <duration
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/numberfield"
                        fieldDescription="Length in minutes"
                        fieldLabel="Duration (minutes)"
                        min="1"
                        name="./duration"
                        step="1"/>
                    <courseLink
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                        fieldDescription="A page inside /content/trainingportal"
                        fieldLabel="Course page"
                        name="./courseLink"
                        rootPath="/content/trainingportal"/>
                    <courseImage
                        jcr:primaryType="nt:unstructured"
                        sling:resourceType="granite/ui/components/coral/foundation/form/pathfield"
                        fieldDescription="An image below /content/dam/trainingportal"
                        fieldLabel="Image"
                        name="./courseImage"
                        rootPath="/content/dam/trainingportal"/>
                </items>
            </column>
        </items>
    </content>
</jcr:root>
```

The new field is `./courseImage`, a pathfield restricted to the training DAM folder. The model gains:


**Source: `modules/l10-dam-image/core/src/main/java/com/trainingportal/core/models/CourseCardModel.java`**

```java
package com.trainingportal.core.models;

import javax.annotation.PostConstruct;

import org.apache.sling.api.SlingHttpServletRequest;
import org.apache.sling.models.annotations.DefaultInjectionStrategy;
import org.apache.sling.models.annotations.Exporter;
import org.apache.sling.models.annotations.Model;
import org.apache.sling.models.annotations.injectorspecific.ValueMapValue;

/**
 * Sling Model behind the Course Card component.
 *
 * <ul>
 *   <li>{@code adaptables = SlingHttpServletRequest}: usable from HTL with {@code data-sly-use}.</li>
 *   <li>{@code resourceType}: lets the Jackson exporter serve {@code .model.json} for the component.</li>
 *   <li>Everything an author can type is validated here, never trusted.</li>
 * </ul>
 */
@Model(
    adaptables = SlingHttpServletRequest.class,
    defaultInjectionStrategy = DefaultInjectionStrategy.OPTIONAL,
    resourceType = CourseCardModel.RESOURCE_TYPE)
@Exporter(name = "jackson", extensions = "json")
public class CourseCardModel {

    public static final String RESOURCE_TYPE = "trainingportal/components/coursecard";

    /** Beginner up to and including this many minutes, Intermediate above it. */
    static final int BEGINNER_MAX_MINUTES = 60;

    private static final String SITE_PREFIX = "/content/trainingportal/";
    private static final String DAM_PREFIX = "/content/dam/trainingportal/";

    @ValueMapValue
    private String title;

    @ValueMapValue
    private String description;

    @ValueMapValue
    private Integer duration;

    @ValueMapValue
    private String courseLink;

    @ValueMapValue
    private String courseImage;

    private String level;

    @PostConstruct
    protected void init() {
        if (duration == null || duration <= 0) {
            level = null;
        } else {
            level = duration <= BEGINNER_MAX_MINUTES ? "Beginner" : "Intermediate";
        }
    }

    public String getTitle() {
        return title;
    }

    public String getDescription() {
        return description;
    }

    public Integer getDuration() {
        return duration;
    }

    /** "45 minutes", or null when no valid duration was authored. */
    public String getDurationLabel() {
        return level == null ? null : duration + " minutes";
    }

    /** "Beginner" (up to 60 minutes) or "Intermediate"; null without a valid duration. */
    public String getLevel() {
        return level;
    }

    /** Lower-case form of the level, safe as a CSS modifier. */
    public String getLevelKey() {
        return level == null ? null : level.toLowerCase(java.util.Locale.ROOT);
    }

    /** Link to a page inside the site (".html" added when missing); null for anything else. */
    public String getLinkUrl() {
        if (courseLink == null || !courseLink.startsWith(SITE_PREFIX) || courseLink.contains("..")) {
            return null;
        }
        String lastSegment = courseLink.substring(courseLink.lastIndexOf('/') + 1);
        return lastSegment.contains(".") ? courseLink : courseLink + ".html";
    }

    /** DAM path of the image, or null when missing or not inside the DAM. */
    public String getImageSrc() {
        return isDamPath(courseImage) ? courseImage : null;
    }

    public boolean isConfigured() {
        return title != null && !title.trim().isEmpty();
    }

    static boolean isDamPath(String path) {
        return path != null && path.startsWith(DAM_PREFIX) && !path.contains("..");
    }
}
```


**Source: `modules/l10-dam-image/ui.apps/src/main/content/jcr_root/apps/trainingportal/components/coursecard/coursecard.html`**

```html
<!--/*
  Version 3 (Lab 10): adds the DAM image. The model returns null for a missing image or for any path
  outside /content/dam/trainingportal, so the <img> is simply not rendered in those cases.
  Production sites should prefer the Core Image component (responsive renditions, WebP, lazy loading).
*/-->
<sly data-sly-use.card="com.trainingportal.core.models.CourseCardModel"
     data-sly-use.template="core/wcm/components/commons/v1/templates.html"/>
<article data-sly-test="${card.configured}" class="tp-course-card">
    <img data-sly-test="${card.imageSrc}"
         class="tp-course-card__image"
         src="${card.imageSrc @ context='uri'}"
         alt="${card.title}"
         loading="lazy"/>
    <div class="tp-course-card__body">
        <h3 class="tp-course-card__title">${card.title}</h3>
        <p class="tp-course-card__meta">
            <span data-sly-test="${card.level}"
                  class="tp-course-card__level tp-course-card__level--${card.levelKey @ context='styleToken'}">${card.level}</span>
            <span data-sly-test="${card.durationLabel}">${card.durationLabel}</span>
        </p>
        <p data-sly-test="${card.description}" class="tp-course-card__description">${card.description}</p>
        <a data-sly-test="${card.linkUrl}" class="tp-course-card__link" href="${card.linkUrl}">View course</a>
    </div>
</article>
<sly data-sly-call="${template.placeholder @ isEmpty=!card.configured}"/>
```

3. `(cd core && mvn clean test)`; deploy `core` and `ui.apps`.
4. Open a card's dialog, choose the image, save: the image shows with `alt` = the card title.
5. **Missing image:** clear the field: no broken `<img>` is rendered.
6. **Foreign path:** in CRXDE set `courseImage` on a card to `/content/dam/other/x.jpg` (any path outside the training folder): no image is rendered. The model, not the template, enforces this.

### Note on alt text and production
`alt` equals the title here, which is acceptable for a teaser. A decorative image should have an empty `alt`. For production prefer the **Core Image** component (responsive renditions, lazy loading, WebP) or delegate to it.

### Verify
```bash
verify.sh l10
```

---

## Lab 11. Scheduler, service user and logging (45 min)

**Objective:** a scheduled job that reads content as a least-privilege service user and writes to its own log.

### Talk (10 min)

- A `Runnable` service with a `scheduler.expression` (Quartz cron) is scheduled by the Sling scheduler. **No expression, no schedule**, so the job is **off on publish** unless a publish config provides one.
- `scheduler.concurrent=false` prevents overlapping runs **on this instance**. It is **not** a cluster-wide singleton; for that add `scheduler.runOn=SINGLE` to the configuration.
- **Never run background code as admin.** A *service user* is a system user with only the permissions it needs, mapped to your bundle through a **sub-service name**. It is created from code with **repoinit**.
- Always **close** a service `ResourceResolver` (try-with-resources).

### Steps

```bash
apply-module.sh l11 <project>
```


**Source: `modules/l11-scheduler/core/src/main/java/com/trainingportal/core/schedulers/CourseScheduler.java`**

```java
package com.trainingportal.core.schedulers;

import java.util.Collections;
import java.util.Map;
import java.util.regex.Pattern;

import org.apache.sling.api.resource.LoginException;
import org.apache.sling.api.resource.Resource;
import org.apache.sling.api.resource.ResourceResolver;
import org.apache.sling.api.resource.ResourceResolverFactory;
import org.osgi.service.component.annotations.Activate;
import org.osgi.service.component.annotations.Component;
import org.osgi.service.component.annotations.Modified;
import org.osgi.service.component.annotations.Reference;
import org.osgi.service.metatype.annotations.AttributeDefinition;
import org.osgi.service.metatype.annotations.Designate;
import org.osgi.service.metatype.annotations.ObjectClassDefinition;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.trainingportal.core.services.CourseService;

/**
 * Scheduled job: counts the course pages and logs the figure next to the configured maximum.
 *
 * <ul>
 *   <li>Picked up by the Sling scheduler because it is a {@code Runnable} service with a
 *       {@code scheduler.expression} (a Quartz cron) in its configuration.</li>
 *   <li>Off unless a run-mode configuration provides the cron: publish never runs it.</li>
 *   <li>Reads the repository as the <b>service user</b> mapped to sub-service
 *       {@code trainingportal-reader} (read-only), never as admin, and always closes the resolver.</li>
 *   <li>{@code scheduler.concurrent=false} stops overlapping runs on THIS instance. It is not a
 *       cluster-wide singleton; for that, add {@code scheduler.runOn=SINGLE} to the configuration.</li>
 * </ul>
 */
@Component(service = Runnable.class)
@Designate(ocd = CourseScheduler.Config.class)
public class CourseScheduler implements Runnable {

    static final String SUB_SERVICE = "trainingportal-reader";
    private static final String DEFAULT_COURSES_PATH = "/content/trainingportal/us/en/courses";
    private static final Pattern SAFE_PATH = Pattern.compile("^/content/trainingportal(/[A-Za-z0-9_-]+)*$");

    private static final Logger LOG = LoggerFactory.getLogger(CourseScheduler.class);

    @ObjectClassDefinition(name = "Training Portal - Course Scheduler",
        description = "Counts course pages on a schedule")
    public @interface Config {

        @AttributeDefinition(name = "Cron expression", description = "Quartz cron; default every 5 minutes")
        String scheduler_expression() default "0 0/5 * * * ?";

        @AttributeDefinition(name = "Allow concurrent executions")
        boolean scheduler_concurrent() default false;

        @AttributeDefinition(name = "Courses path", description = "Must be inside /content/trainingportal")
        String coursesPath() default DEFAULT_COURSES_PATH;
    }

    @Reference
    private ResourceResolverFactory resolverFactory;

    @Reference
    private CourseService courseService;

    private volatile String coursesPath = DEFAULT_COURSES_PATH;

    @Activate
    @Modified
    protected void configure(Config config) {
        coursesPath = isSafe(config.coursesPath()) ? config.coursesPath() : DEFAULT_COURSES_PATH;
        LOG.info("CourseScheduler configured: cron={}, coursesPath={}", config.scheduler_expression(), coursesPath);
    }

    @Override
    public void run() {
        Map<String, Object> auth = Collections.singletonMap(ResourceResolverFactory.SUBSERVICE, SUB_SERVICE);
        try (ResourceResolver resolver = resolverFactory.getServiceResourceResolver(auth)) {
            Resource courses = resolver.getResource(coursesPath);
            if (courses == null) {
                LOG.warn("CourseScheduler: courses path does not exist or is not readable by the service user");
                return;
            }
            int pages = 0;
            for (Resource child : courses.getChildren()) {
                if ("cq:Page".equals(child.getResourceType())) {
                    pages++;
                }
            }
            LOG.info("CourseScheduler: {} course page(s) under {} (configured maximum: {})",
                pages, coursesPath, courseService.getMaximumCourses());
        } catch (LoginException e) {
            LOG.error("CourseScheduler could not open a service resolver for sub-service '{}'. "
                + "Check the service user mapping and that the user exists.", SUB_SERVICE, e);
        }
    }

    static boolean isSafe(String path) {
        return path != null && SAFE_PATH.matcher(path).matches();
    }
}
```

Configuration files:


**Source: `modules/l11-scheduler/ui.config/src/main/content/jcr_root/apps/trainingportal/osgiconfig/config/org.apache.sling.serviceusermapping.impl.ServiceUserMapperImpl.amended~trainingportal.cfg.json`**

```json
{
  "user.mapping": [
    "com.trainingportal.core:trainingportal-reader=[trainingportal-reader]"
  ]
}
```

*Reads: "code in bundle `com.trainingportal.core`, asking for sub-service `trainingportal-reader`, becomes system user `trainingportal-reader`".* **The text before the colon must equal the symbolic name you wrote down in Lab 1.** Edit it if yours differs.


**Source: `modules/l11-scheduler/ui.config/src/main/content/jcr_root/apps/trainingportal/osgiconfig/config/org.apache.sling.jcr.repoinit.RepositoryInitializer~trainingportal.cfg.json`**

```json
{
  "scripts": [
    "create service user trainingportal-reader",
    "set ACL for trainingportal-reader\n  allow jcr:read on /content/trainingportal\nend"
  ]
}
```


**Source: `modules/l11-scheduler/ui.config/src/main/content/jcr_root/apps/trainingportal/osgiconfig/config/org.apache.sling.commons.log.LogManager.factory.config~trainingportal.cfg.json`**

```json
{
  "org.apache.sling.commons.log.names": ["com.trainingportal"],
  "org.apache.sling.commons.log.level": "debug",
  "org.apache.sling.commons.log.file": "logs/trainingportal.log",
  "org.apache.sling.commons.log.additiv": "false"
}
```


**Source: `modules/l11-scheduler/ui.config/src/main/content/jcr_root/apps/trainingportal/osgiconfig/config.author/com.trainingportal.core.schedulers.CourseScheduler.cfg.json`**

```json
{
  "scheduler.expression": "0 0/5 * * * ?",
  "scheduler.concurrent": false,
  "coursesPath": "/content/trainingportal/us/en/courses"
}
```

(The log property really is spelled `additiv`.) The scheduler configuration sits in **`config.author`**, so only author runs the job.

1. **Required content setup:** in Sites, under `/content/trainingportal/us/en/courses`, create two **child pages** with names `aem-fundamentals` and `component-development` using the Course Page template. Verify each is a `cq:Page` in CRXDE. These are separate from the two Course Card components created in Lab 5. The scheduler counts immediate child **pages**, not cards.
2. Deploy `core`, then `ui.config`.
3. `/system/console/slinglog` shows the `com.trainingportal` logger; `crx-quickstart/logs/trainingportal.log` now exists.
4. `tail -f crx-quickstart/logs/trainingportal.log`. After the next scheduled run (up to five minutes): `CourseScheduler: 2 course page(s) under /content/trainingportal/us/en/courses (configured maximum: 5)`.
   To see it sooner, set the cron in configMgr (**Training Portal - Course Scheduler**) to `0/30 * * * * ?` and Save.
5. Confirm the service user: *Tools → Security → Users* (system users), or `verify.sh l11`. It has **read** access to `/content/trainingportal` only.
6. **Break it on purpose:** change the bundle name in the mapping, redeploy `ui.config`. The log shows the `LoginException` message naming the sub-service. Fix it.

> The repoinit ACL needs `/content/trainingportal` to exist, which it does after Lab 1. If the user is missing, look for `RepositoryInitializer` errors in `error.log` and **Save** the repoinit configuration again in configMgr to re-run it.

### Verify
```bash
AEM_LOG_DIR=<aem-folder>/crx-quickstart/logs verify.sh l11
```

---

## Lab 12. Workflow (55 min)

**Objective:** a Java process step in a model with a participant step, run on a page, with history.

### Talk (10 min)

A workflow **model** is a sequence of **steps**. A **Process Step** runs your `WorkflowProcess` (selected by its `process.label`); a **Participant Step** creates an Inbox task for a user or group. The step keeps results in **workflow metadata** (`MetaDataMap`), where later steps (an OR split, a participant) can read them. Models are edited, then **Synced** to the runtime.

Design choice here: the step **only reads** the page and records a verdict. It never changes content, so a failure cannot corrupt a page, and the workflow session's permissions are the user's own.

### The code

```bash
apply-module.sh l12 <project>
```


**Source: `modules/l12-workflow/core/src/main/java/com/trainingportal/core/workflows/CourseValidator.java`**

```java
package com.trainingportal.core.workflows;

import java.util.ArrayList;
import java.util.List;

/**
 * The validation rules, kept free of any AEM API so they can be unit-tested in milliseconds.
 * The workflow step only fetches the values and reports the result.
 */
public final class CourseValidator {

    public static final String SITE_PREFIX = "/content/trainingportal/";
    public static final int MAX_DESCRIPTION_LENGTH = 160;

    private CourseValidator() {
    }

    /** True for a page path inside the Training Portal site (no traversal). */
    public static boolean isPayloadAllowed(String path) {
        return path != null && path.startsWith(SITE_PREFIX) && !path.contains("..");
    }

    /** @return the list of problems; empty means the page is valid */
    public static List<String> validate(String title, String description) {
        List<String> problems = new ArrayList<>();
        if (title == null || title.trim().isEmpty()) {
            problems.add("Title is missing");
        }
        if (description == null || description.trim().isEmpty()) {
            problems.add("Description is missing");
        } else if (description.length() > MAX_DESCRIPTION_LENGTH) {
            problems.add("Description is longer than " + MAX_DESCRIPTION_LENGTH + " characters");
        }
        return problems;
    }
}
```


**Source: `modules/l12-workflow/core/src/main/java/com/trainingportal/core/workflows/CourseValidationProcess.java`**

```java
package com.trainingportal.core.workflows;

import java.util.List;

import org.apache.sling.api.resource.Resource;
import org.apache.sling.api.resource.ResourceResolver;
import org.apache.sling.api.resource.ValueMap;
import org.osgi.service.component.annotations.Component;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.adobe.granite.workflow.WorkflowException;
import com.adobe.granite.workflow.WorkflowSession;
import com.adobe.granite.workflow.exec.WorkItem;
import com.adobe.granite.workflow.exec.WorkflowData;
import com.adobe.granite.workflow.exec.WorkflowProcess;
import com.adobe.granite.workflow.metadata.MetaDataMap;

/**
 * A workflow Process Step. Appears in the model editor under the label below.
 *
 * <p>It does NOT modify content. It validates the page and records the verdict in the workflow
 * metadata ({@code validationResult} = valid | invalid, {@code validationProblems}) so that a
 * later OR-split or participant step can act on it. Running with the workflow session means the
 * user's own permissions apply.</p>
 */
@Component(
    service = WorkflowProcess.class,
    property = { "process.label=Training Portal Course Validation" })
public class CourseValidationProcess implements WorkflowProcess {

    static final String RESULT_KEY = "validationResult";
    static final String PROBLEMS_KEY = "validationProblems";

    private static final Logger LOG = LoggerFactory.getLogger(CourseValidationProcess.class);

    @Override
    public void execute(WorkItem item, WorkflowSession session, MetaDataMap args) throws WorkflowException {
        MetaDataMap workflowMeta = item.getWorkflow().getWorkflowData().getMetaDataMap();
        WorkflowData data = item.getWorkflowData();

        if (!"JCR_PATH".equals(data.getPayloadType())) {
            finish(workflowMeta, "invalid", "Payload is not a repository path");
            return;
        }
        String path = String.valueOf(data.getPayload());
        if (!CourseValidator.isPayloadAllowed(path)) {
            finish(workflowMeta, "invalid", "Payload is outside the Training Portal site");
            return;
        }

        ResourceResolver resolver = session.adaptTo(ResourceResolver.class);
        Resource content = resolver == null ? null : resolver.getResource(path + "/jcr:content");
        if (content == null) {
            finish(workflowMeta, "invalid", "Page has no content node");
            return;
        }

        ValueMap props = content.getValueMap();
        List<String> problems = CourseValidator.validate(
            props.get("jcr:title", String.class), props.get("jcr:description", String.class));
        finish(workflowMeta, problems.isEmpty() ? "valid" : "invalid", String.join("; ", problems));
        LOG.info("Course validation finished: {}", workflowMeta.get(RESULT_KEY, String.class));
    }

    private static void finish(MetaDataMap meta, String result, String problems) {
        meta.put(RESULT_KEY, result);
        meta.put(PROBLEMS_KEY, problems);
    }
}
```


**Source: `modules/l12-workflow/core/src/main/java/com/trainingportal/core/workflows/package-info.java`**

```java
@org.osgi.annotation.versioning.Version("1.0.0")
package com.trainingportal.core.workflows;
```


**Source: `modules/l12-workflow/core/src/test/java/com/trainingportal/core/workflows/CourseValidatorTest.java`**

```java
package com.trainingportal.core.workflows;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.jupiter.api.Test;

class CourseValidatorTest {

    @Test
    void completePageIsValid() {
        assertTrue(CourseValidator.validate("AEM Fundamentals", "Learn the basics").isEmpty());
    }

    @Test
    void missingTitleAndDescriptionAreBothReported() {
        assertEquals(2, CourseValidator.validate(" ", null).size());
    }

    @Test
    void overlongDescriptionIsRejected() {
        String longText = "x".repeat(CourseValidator.MAX_DESCRIPTION_LENGTH + 1);
        assertEquals(1, CourseValidator.validate("t", longText).size());
        assertTrue(CourseValidator.validate("t", "x".repeat(CourseValidator.MAX_DESCRIPTION_LENGTH)).isEmpty());
    }

    @Test
    void onlyPagesInsideTheSiteAreAccepted() {
        assertTrue(CourseValidator.isPayloadAllowed("/content/trainingportal/us/en/home"));
        assertFalse(CourseValidator.isPayloadAllowed("/content/other/home"));
        assertFalse(CourseValidator.isPayloadAllowed("/content/trainingportal/../etc"));
        assertFalse(CourseValidator.isPayloadAllowed(null));
    }
}
```

The rules sit in `CourseValidator` (no AEM API), so they are unit-tested in milliseconds; the process step only fetches values.

### Steps

1. `(cd core && mvn clean test)`, then deploy `core`.
2. `/system/console/components`: `CourseValidationProcess` is **active**.
3. *Tools → Workflow → Models → Create → Create Model*: title **Training Portal Course Approval**. **Open** it.
4. Delete the default step if present, then drag in a **Process Step**. Configure: **Process** = *Training Portal Course Validation*, tick **Handler Advance**.
5. Add a **Participant Step** after it. Title *Review course page*, **Participant** = your user (or `administrators`). Save.
6. Click **Sync** (or *Sync Model*) so the runtime sees the model.
7. **Required precondition:** open the **Courses page properties** (not a Course Card dialog). Set a non-empty page **Title** and **Description**. In CRXDE confirm `jcr:title` and `jcr:description` exist at `/content/trainingportal/us/en/courses/jcr:content`. The workflow reads these page properties, not Course Card fields.
8. Start it: back in *Models*, select the model → **Start Workflow**, **Payload** = `/content/trainingportal/us/en/courses`. Start.
9. *Inbox* (the bell icon): open the task and **Complete** it.
10. *Tools → Workflow → Instances* (running) and **Archive** (finished): open the instance and read the history.
11. To see the verdict, open the instance in CRXDE under `/var/workflow/instances/…` and read the workflow metadata: `validationResult` = `valid`, or `invalid` with `validationProblems`. (the exact folder under `/var/workflow` varies by build)
12. **Make it fail:** start the workflow on a page with no description. The verdict is `invalid` and says why.

### Stretch (15 min)
**Optional; instructor must verify ECMA bindings on the installed SP2 build.** Add an **OR Split** after the process step. Branch 1 rule (ECMA script): `function check() { return workflowData.getMetaDataMap().get("validationResult") == "valid"; }`. Branch 2 leads to a participant step **Fix page**. (the script bindings available in an OR split, such as `workflowData`, can differ slightly by build.) Valid pages go straight to approval; invalid ones are routed back to an author.

> Process-step label and model synchronisation must be re-verified after every deployment that changes either.

### Verify
```bash
verify.sh l12
```
Model and history are checked by hand in the UI.

---

## Lab 13. Testing and the debugging toolbox (40 min)

**Objective:** run the whole test suite, then diagnose seven deliberately injected faults with the right tool.

### Part 1: full build and checklist (10 min)

```bash
mvnd clean install -PautoInstallSinglePackage
verify.sh all
```

Confirm: package installed · core bundle Active · pages load · header and footer present · cards authored · model values shown · CSS and JS loaded · style selection works · OSGi config applied · servlet JSON valid · scheduler logged · workflow completed.

Unit tests in this kit: `CourseCardModelTest` (AEM Mocks: null, 0, 45, 60, 61, 120 minutes; link and image rules), `CourseServiceImplTest`, `CourseValidatorTest`.

### Part 2: the toolbox (5 min)

| Tool | Answers |
|---|---|
| `crx-quickstart/logs/error.log`, `trainingportal.log` | what happened, with stack traces |
| `/system/console/bundles` | is my bundle Active? which import is missing? |
| `/system/console/components` | is my DS component active? **unsatisfied reference**? |
| `/system/console/configMgr` | which configuration is live? |
| `/system/console/status-slingmodels` | is my model registered, and for which resource type? |
| `/system/console/servletresolver`, `/jcrresolver`, `/requests` | which script or servlet handles this URL? |
| CRXDE Lite | what is really stored? |
| `?wcmmode=disabled`, `?debugClientLibs=true` | published markup, raw clientlibs |
| Browser dev tools (Network, Elements) | which CSS and JS loaded; what DOM and wrapper |

### Part 3: break and fix (25 min)

Pairs. One person injects a fault from the list, the other diagnoses using only the toolbox. Swap.

| # | Fault to inject | Symptom | Where to look |
|---|---|---|---|
| 1 | Remove `<classifier>apis</classifier>` | build errors, "package does not exist" | Maven output |
| 2 | Change the bundle name in the user mapping | `LoginException` in the scheduler log | `trainingportal.log`, Bundles |
| 3 | Replace `javax.annotation.PostConstruct` with `jakarta.annotation.PostConstruct` in the model | `getLevel()` is always null, no error | Compare to Lab 6; read A2 |
| 4 | Set `allowProxy` to `false` | styles missing from the proxy URL | Network tab, `dumplibs` |
| 5 | Typo in the servlet's resource type | 404 on the JSON URL | `servletresolver`, `components` |
| 6 | Change the page resource type used in the servlet to `cq:Page` or remove `/jcr:content` from the URL | 404 | Lab 2, `jcrresolver` |
| 7 | Point repoinit at a path that does not exist | service user missing | `error.log` |

---

## Lab 14. Packages, replication, Dispatcher and a security review (25 min)

**Objective:** capture your authored content, understand what travels to publish, and review the code you wrote with a security checklist.

### Step 1: the golden package (8 min)
Content you authored (template, pages, DAM) lives in the repository, not in Git.

1. `/crx/packmgr` → **Create Package**: group `trainingportal`, name `trainingportal-golden-content`.
2. **Edit → Filters**: `/content/trainingportal`, `/conf/trainingportal`, `/content/dam/trainingportal`.
3. **Build**, then **Download**. This is your restorable snapshot and the catch-up for the next class. Templates, policies, site structure and workflow models are created through the AEM UI and should then be exported into source control; this is how. (The workflow model lives under `/conf/global/settings/workflow/models`: add that filter for the model.)

### Step 2: replication (7 min)
Author **activates** content; **replication agents** ship it to publish. **Code (`/apps`) is not published from author**: it is deployed to each tier by the same package. Contrast mutable content (packages or replication) with immutable code (the build).

### Step 3: Dispatcher in 5 minutes
An *illustrative* deny-by-default filter (use your team's baseline, not this snippet):

```
/filter {
  /0001 { /type "deny"  /url "*" }
  /0010 { /type "allow" /method "GET" /url "/content/trainingportal/*" }
  /0020 { /type "allow" /method "GET" /url "/etc.clientlibs/trainingportal/*" }
  /0030 { /type "allow" /method "GET" /url "/content/dam/trainingportal/*" }
  /0090 { /type "deny"  /url "/system/*" }
  /0091 { /type "deny"  /url "/crx/*" }
  /0092 { /type "deny"  /url "*.infinity.json*" }
  /0093 { /type "deny"  /url "*.courseinfo.json*" }
}
```
**Security lab prerequisite:** A live Dispatcher and publish tier are not part of the default author-only setup. Treat this filter as a discussion example only, not a deployable configuration. On an environment with Dispatcher, confirm that anonymous GET requests to `/system/console`, `/crx/de`, `.infinity.json`, and `.courseinfo.json` are blocked; test ordinary pages and `/etc.clientlibs` remain accessible. Resource-type servlet registration alone is not an authorization boundary.

Tie back: Lab 7 (clientlibs allowed), Lab 9 (the endpoint is **denied** on publish), Lab 2 (`.infinity.json` denied).

### Step 4: review your own code (5 min)
Tick each, and find the line that proves it.

- [ ] Servlet is resource-type bound, GET only, selector and extension set, refuses anonymous users.
- [ ] Every author-supplied value is validated in the model (link and image paths; difficulty-style allow-lists).
- [ ] No `context='unsafe'` in any HTL file.
- [ ] No admin resolver: a service user with read-only ACL, created with repoinit.
- [ ] Every opened `ResourceResolver` is closed.
- [ ] Configuration values are clamped.
- [ ] No secrets in Git or configs.
- [ ] Logic in Sling Models with unit tests; `javax.annotation.PostConstruct`.
- [ ] Clientlibs use `allowProxy=true`; no inline scripts.
- [ ] Rejected input is logged **without echoing it**.



---

# Part D: Assessment

## Lab 15. Assessment: build, demo, quiz and retrospective (70 min)

### 1. Rubric (100 points)

Passing score **75**, **and** a successful build, **and** a functional Course Card. Evidence is shown live; `verify.sh` gives a first pass.

| Area | Pts | Evidence the learner shows | Automated check |
|---|---|---|---|
| Project generation and deployment | 10 | `BUILD SUCCESS`, package installed, bundle Active, `/apps/trainingportal` in CRXDE | `verify.sh l01` |
| Site, template, header, footer | 15 | three pages from the Course Page template; header with menu and footer on all pages | `l03`, `l04` |
| Dialog and component | 15 | a working authoring dialog; two cards saved with properties visible in CRXDE | `l05` |
| Sling Model and HTL | 15 | Beginner/Intermediate badge; unit tests green; `.model.json` | `l06` |
| Clientlibs and styles | 10 | stylesheet in the Network tab via `/etc.clientlibs`; Featured/Compact switch in the editor | `l07` |
| OSGi service and servlet | 10 | config changed 10 to 5 in configMgr; servlet JSON shows 5; anonymous request refused | `l08`, `l09` |
| Scheduler | 10 | `CourseScheduler:` line in `trainingportal.log`; service user exists | `l11` |
| Workflow | 10 | model synced; instance in Archive; process history and Inbox task | `l12` (partial) + UI |
| Debugging and explanation | 5 | diagnoses one injected fault and explains the service-user and javax decisions | oral |

### 2. Demo script (10 min per learner or pair)

Show, in this order: a working page URL · the Course Card dialog · `.model.json` or the level badge · the servlet response · the scheduler log line · the workflow history · one break-and-fix.

### 3. Written quiz (15 min; answers at the end)

1. Which API jar coordinate and classifier does AEM 6.5 LTS need for compilation?
2. `javax` or `jakarta` for annotations in LTS Sling Models? What happens if you use the wrong one?
3. A servlet bound to `home.courseinfo.json` returns 404. What is the most likely cause?
4. Where does AEM store a component's authored properties?
5. Why does the Style System CSS use `.cmp-featured .tp-course-card`?
6. What does `allowProxy=true` do, and what breaks without it?
7. Why must a scheduler use a service user, and what must you do with its resolver?
8. What does `scheduler.concurrent=false` guarantee, and what does it not?
9. Which part of an editable template is inherited live by existing pages: structure or initial content?
10. Name two things the Course Card model validates and why.

**Answers.** (1) `com.adobe.aem:uber-jar:6.6.0`, classifier `apis`. (2) `javax`; a `jakarta` annotation is silently ignored. (3) The URL targets the `cq:Page` node; use `…/jcr:content.courseinfo.json`, or the resource type does not match. (4) On the component node under the page's `jcr:content` tree, e.g. `…/root/container/container/coursecard`. (5) AEM adds the style class to a wrapper `div` around the component. (6) Serves `/apps` clientlibs via `/etc.clientlibs`; without it they 404 on publish. (7) Least privilege; close it (try-with-resources). (8) No overlapping runs on this instance; it is not cluster-wide (use `scheduler.runOn=SINGLE`). (9) Structure. (10) Link inside the site, image inside the DAM folder (no javascript: or foreign URLs); duration non-positive means no level.

### 4. Retrospective (10 min)
Each person writes one *start*, one *stop*, one *continue*. Then discuss next steps: Experience Fragments for header and footer, Core Navigation and Core Image, integration tests in `it.tests`, a CI pipeline, and moving authored content into source control.

---

# Appendices

## Appendix A: URL cheat sheet (author `http://localhost:4502`)

`preflight.sh` checks the rows marked ✔. Paths can differ slightly between builds; confirm them in the UI.

| Purpose | URL | |
|---|---|---|
| Start | `/aem/start.html` | ✔ |
| Sites | `/sites.html/content` | ✔ |
| Assets | `/assets.html/content/dam` | ✔ |
| Page editor | `/editor.html/content/trainingportal/us/en.html` | |
| Published view | add `?wcmmode=disabled` | |
| Templates | `/libs/wcm/core/content/sites/templates.html/conf` | ✔ |
| Content Fragment Models | `/libs/dam/cfm/models/console/content/models.html/conf` | ✔ |
| Workflow Models | `/libs/cq/workflow/admin/console/content/models.html` | ✔ |
| CRXDE Lite | `/crx/de/index.jsp` | ✔ |
| Package Manager | `/crx/packmgr/index.jsp` | ✔ |
| OSGi bundles | `/system/console/bundles` | ✔ |
| OSGi configuration | `/system/console/configMgr` | ✔ |
| OSGi components | `/system/console/components` | ✔ |
| Sling log support | `/system/console/slinglog` | ✔ |
| Sling Models | `/system/console/status-slingmodels` | ✔ |
| Servlet resolver | `/system/console/servletresolver` | ✔ |
| JCR resolver | `/system/console/jcrresolver` | ✔ |
| Recent requests | `/system/console/requests` | ✔ |
| Product info | `/system/console/status-productinfo` | |
| Client library dump | `/libs/granite/ui/content/dumplibs.html` | ✔ |
| Debug clientlibs | add `?debugClientLibs=true` | |

## Appendix B: Troubleshooting matrix {: .flow}

| Symptom | Likely cause | Check |
|---|---|---|
| "package … does not exist" everywhere | missing `apis` classifier | rerun the patch (Lab 1) |
| Bundle Installed, not Active | unresolved import | Bundles, then *Imported packages*; `jakarta.*` and Guava are typical |
| Model values null, no error | `jakarta.annotation.PostConstruct` | switch to `javax` |
| Model not found in HTL | package not exported | `package-info.java` with `@Version` |
| Component missing in sidebar | group not allowed by the policy | Lab 3 container policy |
| Styles missing | clientlib not included or `allowProxy` off | Network tab, Lab 7 |
| Servlet 404 | `jcr:content` missing in the URL, resource-type mismatch | Labs 2 and 9 |
| `LoginException` in scheduler | mapping bundle name wrong, user missing | Lab 11 |
| Service user absent | repoinit failed | `error.log`; Save the config again |
| `verify.sh` says 401 | wrong `AEM_PASS` | re-export it |

If a learner is stuck for more than 5 minutes: `apply-module.sh <lab> <project>`, redeploy, `verify.sh`, move on.

## Appendix C: Kit contents {: .flow}

```
trainingportal-lts-lab-kit/
  README.md
  modules/l04-header-footer/ ... l12-workflow/    finished files per lab (project layout)
  preflight.sh  verify.sh  apply-module.sh  patch-archetype-for-lts.py  lib.sh
```
Set up once: `export KIT=~/trainingportal-lts-lab-kit; export PATH="$KIT/scripts:$PATH"`.

## Appendix D: Optional extensions {: .flow}

| Extension | What it adds |
|---|---|
| Inject `CourseService` into the card model | `@OSGiService` (Lab 8) |
| Header and footer as **Experience Fragments** | the production pattern |
| Core **Navigation** instead of `SiteNavModel` | production menu |
| Core **Image** component | responsive renditions instead of a raw `<img>` |
| **Query Builder** course search | a searchable course list driven by predicates |
| **Overlay** vs **proxy** and the Sling Resource Merger | extend or shadow product components safely |
| **Internationalisation** | dictionary-based translation of labels (Sling i18n) |

## Appendix E: Glossary {: .flow}

| Term | Meaning |
|---|---|
| JCR / Oak | the content repository API and its implementation |
| Sling | the web framework mapping URLs to resources and resources to scripts |
| OSGi / bundle / DS | the module system / a Java module / the annotation-based component model |
| HTL | server-side template language with context-aware escaping |
| Resource type | a path string telling Sling which script renders a node |
| Proxy component | your component whose `sling:resourceSuperType` points at a Core Component |
| Service user | a system user with minimal permissions for background code |
| Repoinit | declarative script that creates users, ACLs and paths at startup |
| Run mode | environment label (`author`, `publish`, `dev`) that selects configurations |
| Policy | reusable component rules attached to an editable template |
| Style System | author-selectable CSS classes defined in a policy |
