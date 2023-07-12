## Current framework
- The documentation is outdated, ambiguous, partial, and scattered. It exists in sources such as:
  - `greenstand.org/docs` which is redundant and can be removed
  - `greenstand.org/devbox` which needs to be reviewed, and then removed
  - In various Gitbook spaces, many which are inaccesible as:
    - their published spaces are not linked anywhere
    - their published spaces are not indexed by google
    - the Greenstand Gitbook is locked behind authorization
  - In repos like :
    - `greenstand.github.io` 
    - `Greenstand-Overview` 
    - `treetracker-decisions` (unsure)
    - `system-design-docs`
    - `treetracker-infrastructure-adr` (unsure)
    - `meta-documentation`
    - `microservices-documentation` (empty, to be removed)
  - In various wikis, as listed in [Current Documentation](currentDocumentation.md)

## New framework
- Single Documentation Portal
  - All the documentation is consolidated under `docs.greenstand.org`, via Gitbook
  - All documentation is up to date
- Following `docPlan.md`
- Keeping a changelog
- All relevant project repos have a `docs/` directory corresponding to a Gitbook space.
- Sustainable processes for maintaining documentation.

## Project Scope
The scope of this documentation project includes:
- Indexing and auditing the status of all existing documentation
- Bringing the documentation into a standardized and editable format and following the DPG standard opensource best practices
- Removing old and irrelevant documentation
- Updating and standardizing all project-critical documentation

## Indexing existing documentation
- Complete inventory of existing documentation
- MD file on Github with links and descriptions of all documentation
- See [Documentation Inventory](documentationInventory.md)

## Standardizing the documentation

![img.png](../assets/img.png)

Using the above image as a guideline, we can divide the documentation into 2 types (for now):
1. User Documentation (End users)
2. Technical/System Documentation (Contributors)
   - We ignore the third party integrations for later

  
- **User documentation**
  - Support Portals
  - FAQs
  - Video tutorials
  - Step-by-step guides
  - Embedded assistance
  - Software requirements.
  - Installation guide.
  - How to start the system.
  - How to use features of a product.
  - Screenshots explaining those features.
  - Contacts of the developer if an undocumented question arises and more.
  > Source: https://medium.com/@kesiparker/technical-documentation-vs-user-documentation-ff68e7de1985
- **Technical documentation**
  - How the review process works 
  - Style guides 
  - Schedules 
  - Design and Implementation
  - Release process checklists
  - Tools to use for development
  - How to use the build process 
  - Coding practices to follow 
  - Process to release a build to QA for testing  
  - Deploy or release a product
  - May use jargon or industry-specific terms
  

## Removing old documentation
## Updating documentation


## TODOs
- Standardize documentation for project repos
  - README.md 
    - Summary
    - Installation steps
    - Contributing guidelines
    - Link to Github space
  - Gitbook space
  - No wikis