# How to Approach Test Planning and Test Data for project

This guideline covers only one part of a broader, in-house framework: Edenlab is building its own AI-assisted methodology across the whole BA → SA → Dev path. The rest of this document zooms into one part of that framework — test planning and test data for project.

- A business analyst (or product owner) owns User Stories as the single entry point for requirements, including "container" stories (e.g. "as a user, I want to log in"); the system analyst adds the detailed layer — business rules, data/FHIR mapping, validations, and links to related artifacts (mockups, API specs).

- The specification handed to development stays minimal and abstract. The heavier system/technical documentation is generated after development, from the actually merged code, and reused as context for the next iteration — it isn't maintained as a large document written up front.

- AI generates the technical forms — FHIR mapping, JSON, regular expressions — from the SA's natural-language detailing, but a human always reviews them: AI's proposals for the data model are a starting point, not a trusted final result.

- Testing runs on two tracks: sanity/happy-path acceptance at the story level (the regression suite), and deep SA-level testing during acceptance of a developer's task. The skills in this guideline mainly serve the second, deeper track.

- This is where this guideline fits in: the test-data/test-plan skills consume the SA's detailing (US, acceptance criteria, business rules, FHIR mapping) as input, and their own generation follows the same "atomic AI steps with a human checkpoint" principle as the framework as a whole.

- If the US or acceptance criteria change after test data/plan generation, regenerate them from the updated source instead of hand-editing the stale output — the same "the source of truth gets recomputed, not patched" principle covered in section 3.3.

The approach to test plans and test data, and how to choose between three Claude skills that automate this work: us-test-data, epic-qa-suite, and test-data-generator.

All three work from User Stories (US) in Confluence and produce test data or QA artifacts, but they differ in scope of work, output format, and how tightly they are tied to project's own FHIR specifications. Use the table below to pick a skill, and when in doubt, read that skill's full section before running it.

The rule that applies to all three: their output is an AI-generated starting point, not a finished deliverable. Every output needs a human review before it is used for real testing.

## 1. Skills overview

| Skill                 | What it does (short)                                                | Output format                                     |
| --------------------- | ------------------------------------------------------------------- | ------------------------------------------------- |
| us-test-data          | Generates FHIR fixtures (test data) only                            | JSON fixtures + curl script + descriptions        |
| epic-qa-suite | Full QA package: Postman collection + test plan + fixtures          | Postman JSON + CSV/XLSX test plan + JSON fixtures |
| test-data-generator   | Generates patient data (optionally FHIR) from a test plan and/or US | New Google Sheet                                  |

## 2. Methodology: how to approach the test plan and test data

### 2.1 Combined vs. split approach to generation

- By default, for project epics the team uses the combined approach (epic-qa-suite): fixtures and the test plan are generated together in one run. The reason is that project epics typically produce many fixtures, and manually re-associating them with test cases after the fact is inconvenient.
- The split approach (us-test-data first, then the test plan as a separate step) remains a valid option — especially for lighter epics with few fixtures, or when you deliberately want to review the data on its own before building scenarios.
- Both approaches are officially recognized as workable; the choice depends on the epic's size and how fixture-heavy it is, not on a hard "always split" rule (this refines an earlier team decision that initially required splitting the steps in every case).

### 2.2 When you must verify that test data is adequate

- Verifying the test suite (including the data) is mandatory at one specific point in the process: the handoff from development to acceptance testing (at the analyst level). Before the test plan is actually used for acceptance, the test data must have been checked.
- Checking the data earlier — right after generation, before development — is fine when there's time or a specific reason, but it is NOT a requirement. The team deliberately does not mandate reviewing test data purely as "a development input": that adds no value if the data will be checked anyway once it's actually used.
- Why: a fixture can break for reasons that have nothing to do with the generator itself (for example, a change a developer committed alongside it) — so the first real check "in battle" is unavoidable either way, and mandating an extra early review "just in case" doesn't add value.

### 2.3 What to do when test data turns out to be incorrect

- If the data itself was generated wrong (the AI made a mistake: the wrong field type, an invalid code, an extra invented property), and there's nothing wrong on the other side (the StructureDefinition and the backend's behavior are both fine) → fix it by changing the generator, not one fixture by hand — the skill/script builder function that produced it — so it emits correct data by code going forward. This is the same rule the skills already apply internally (Step 7 of epic-qa-suite explicitly says: when a mismatch is found, fix the builder function, regenerate, and re-validate — don't hand-patch just the one file); it is now confirmed at the methodology level too.
- If the root cause is a mismatch between the requirements (US/acceptance criteria) and the actual implementation (for example the backend really behaves differently from what's documented, or the FHIR resource's real structure differs from the spec) → fix that mismatch first (clarify the requirements, or agree with development on the correct behavior), and only then regenerate the test data. Regenerating without fixing the root cause is wasted effort — the data will again be "wrong" relative to the system's actual behavior.
- Either way, avoid a one-off manual patch to a fixture as the final fix: beyond an urgent, one-time need (e.g. to unblock testing right now), it doesn't remove the cause, and the next generation will reproduce the same defect.

### 2.4 Static vs. dynamic data: what to reuse and what to regenerate

- Split test data into static (organizations, doctors, medications, patients — expensive or impossible to recreate every time: organizations live on separate tenants, doctors need a real registration flow) and dynamic (consultation notes, prescriptions, dispenses — always created fresh for each scenario).
- The fixture-generation step should check what static data already exists (the roles table, fixtures from earlier epics) and report exactly what's missing, rather than inventing the whole set from scratch every time.
- Whatever the AI cannot create itself (for example a doctor who needs a real email/password registration flow) is created by a human — that's a normal boundary of automation, not a gap in the process.

### 2.5 UI test scenarios must be navigable through the interface

- Prerequisites list only static data and human-usable identifiers ("Doctor X is logged in," "the patient with document ID Y") — never raw JSON or a GUID that can't be found in the interface.
- Dynamic entities are expressed as actions in the scenario's steps ("create a consultation note for this patient, add a medication, click Create prescription"), not as references to pre-generated fixtures.
- Don't artificially over-detail obvious, learn-once actions (how to log in, how to search for a patient) — the same way you wouldn't spell them out for a human tester. The actual problem to fix is UI-unusable references (JSON/GUIDs), not a lack of step granularity.

### 2.6 Completeness of test-case coverage

- Completeness is measured against the US's acceptance criteria, not an arbitrary fixed cap: if the AC describe several distinct negative conditions, all of them must be covered by test cases — even at the sanity-check level.
- An artificial cap on the number of negative cases (e.g. "one negative per boundary") — which is how it used to be done — was recognized by the team as the wrong approach, since it leaves documented logic untested.

### 2.7 Validating fixtures before use

- Every fixture ships with a curl command (it immediately shows which endpoint to call and what body to send) — so even an analyst without deep FHIR knowledge can apply and check the data themselves, instead of reverse-engineering the bundle by hand.
- Validation has two parts, both of which matter: a plain-language description from the AI of exactly which cases are covered (e.g. "active organization / inactive organization"), checked visually; and actually applying the fixture (the curl returns 200/201, and the resource can be found through the UI), wherever that's technically possible.
- JSON correctness is checked at the fixture-generation step itself, not deferred until scenarios run — otherwise it's hard to tell whether a scenario failed because of a bad fixture or a real product defect.

### 2.8 Where these skills sit in the team's wider SDD approach

- The input to these skills is the output of the SA's SDD-style detailing: acceptance criteria, business rules, and the US's FHIR mapping. The quality of the generated data and test plan depends directly on how complete that detailing is — an incomplete US produces an incomplete or invented result, and that's not the skill's fault.
- If the US or acceptance criteria change after test data/plan generation, regenerate them from the updated source instead of hand-editing the stale output. It's the same "the source of truth gets recomputed, not patched" principle the team applies to its other generated artifacts.
- Don't confuse these QA artifacts with a separate step in the team's SDD framework — the system documentation generated from code after development. These are different steps with different purposes: this one prepares for testing; that one documents after the fact.
- Splitting generation into atomic steps with a human checkpoint (as us-test-data / epic-qa-suite do) isn't unique to test data — it's a general principle of the team's SDD framework.

## 3. Detailed skill descriptions

### `us-test-data`

Generates FHIR fixtures (test data) only, for an epic or specific US — without building a test plan or a Postman collection in the same run.

**When to use**

- You only need test data (FHIR fixtures) for a project epic or specific US codes (e.g. RCNP003, DM016.1, P008).
- A request like "generate test data/fixtures for this epic" or "I need seed data for these US" — with no test plan and no Postman collection involved.
- A Confluence epic link or a list of US codes is enough to get started.
- The epic is small or has few fixtures, and you deliberately want to split the work into two steps: first generate and review the test data with this skill, then — as a separate step (e.g. via epic-qa-suite) — build a test plan from the data. This is one of two officially valid team approaches (see section 2.1) — for fixture-heavy project epics the combined skill epic-qa-suite is the typical default.

**When NOT to use (use this instead)**

- You also need a test plan and/or a Postman collection in the same run → use epic-qa-suite instead (it includes this fixtures step and adds a test plan and Postman collection on top).
- You want the data written straight into Google Sheet rows next to test cases, not as separate JSON files → use test-data-generator.
- You need the actual specification text for development (a BE spec) or Given/When/Then acceptance criteria for a backend task, not test data — these are different deliverables: be-spec writes the specification, and ac-gherkin derives acceptance criteria from an already-written specification.

**What to give the prompt as input**

- A Confluence epic link (the page listing the US) OR a list of specific US codes, and which US are actually in scope.
- The path to the local folder with the project's real FHIR StructureDefinitions/CodeSystems/ValueSets — this is the ground truth; say plainly if no such folder is available.
- Whether a canonical set of static entities (organizations, doctors, medications, the roles table) already exists to reuse, or needs to be created from scratch (see section 2.4).
- Whether ready-to-run curl commands are needed to load the fixtures (default: yes), and whether the target environment's base URL is known.
- Fixture format (JSON files is the default) and whether a zip archive is wanted.

**What you get as output**

- FHIR JSON fixtures — one file per resource/scenario, each with a human-findable identifier (a name/ID you could actually search for in the UI), not just a raw GUID.
- load_fixtures.sh — ready-to-run curl commands to load every fixture into the target environment.
- descriptions.md — a plain-language description of each fixture: what it represents and which US/scenario it supports.
- A validation report (PASS/WARN/FAIL per fixture) that states clearly whether only a structural check was done, or also a "live" one (curls actually run against a real environment).

**What to do with the output**

- Review the fixtures before moving on to test scenarios — that's good practice, though formally the check becomes mandatory only at the handoff of the test suite to acceptance testing (see section 2.2).
- Read every WARN in the validation report by hand — don't dismiss them as "just warnings."
- If a test environment is available, run the curl scripts and confirm real 200/201 responses.
- If a fixture turns out to be wrong, identify the cause (a generator error vs. a mismatch between requirements and implementation) and act per section 2.3 — don't treat a manual patch to the file as the final fix.
- Only after reviewing the fixtures, move on to epic-qa-suite (or a separate test-scenario request) if test cases are needed next.
- Never treat the output as "production-ready" without review — it's an AI-generated starting point.

### `epic-qa-suite`

Builds a full QA package for an epic: a Postman collection, a test plan, and FHIR fixtures, cross-referenced with each other.

**When to use**

- You need a full QA package for a project epic: a Postman collection + a test plan (spreadsheet) + FHIR fixtures for UI scenarios that can't be exercised through a documented API.
- A request like "test this epic," "build a test plan for these US," or "I need a Postman collection for this API" — even from just a Confluence link.
- You need traceability: a test-plan row linked to a specific Postman request and/or a specific fixture.
- The epic is fixture-heavy (a lot of test data) — for epics like that, this is the team's typical default on project: generate fixtures and the test plan in one run rather than as separate steps (see section 2.1).

**When NOT to use (use this instead)**

- You only need data/fixtures, no test plan or Postman → us-test-data is enough, and faster — and more convenient for epics with few fixtures.
- You need the actual specification text for development (a BE spec) → be-spec.
- You only need Given/When/Then acceptance criteria for an already-written specification → ac-gherkin.

**What to give the prompt as input**

- An epic link plus which US are in scope (and which are explicitly out of scope).
- The Confluence pages documenting the API specs that cover this epic's US (not every US has a documented API — say which ones do).
- The path to the local FHIR StructureDefinitions/CodeSystems/ValueSets folder (the same ground truth used by us-test-data).
- Whether a static set of entities (organizations/doctors/medications/roles) already exists to reuse (see section 2.4).
- Postman conventions: base URL (hardcoded or a {{baseUrl}} variable), how to represent steps that can't be automated (OTP codes, manually-issued tokens).
- Where the test plan should live: a new file, or an existing Google Sheet (link + tab) — understanding that writing directly into an existing tab may not be technically possible.
- A link to the Users/roles spreadsheet (who can log in as what) and the default tenant/client.
- Test plan depth: sanity check or deep/exhaustive — either way, coverage is measured against the acceptance criteria, with no artificial cap on negative cases (see section 2.6).

**What you get as output**

- A Postman collection (JSON): a folder per endpoint × positive/negative, tests asserting status code and error text, collection variables.
- A test plan CSV + a styled XLSX with columns US #, User Story, TC ID, Scenario, Type, Prerequisites, Steps, Expected result, Postman request/Fixture, Notes, plus a Legend & Scope sheet.
- FHIR fixtures for UI-only scenarios + load_fixtures.sh + descriptions.md + a zip archive.
- An explicit report on which team conventions were actually applied (static-entity reuse, UI-native step wording, fresh entities per scenario, curls) and which weren't, and why.

**What to do with the output**

- Check the cross-references: every TC ID in the test plan points to a real Postman request/fixture, and every fixture/request is referenced by at least one row.
- Check formatting: Prerequisites and Steps are numbered lists; the US # column is sorted in order; UI scenarios are navigable through the interface with no raw JSON/GUIDs in the steps (see section 2.5).
- Run the Postman collection against a test environment if one is available.
- If you asked for a write into an existing Google Sheet and that wasn't technically possible, copy the results in manually from the delivered CSV/XLSX.
- Hand the package to a QA engineer/tester. Formally, verifying the test suite (including the data) becomes mandatory specifically at the handoff from development to acceptance testing — see section 2.2.

### `test-data-generator`

Generates patient data (optionally full FHIR resources) from a Google Sheets test plan and/or Confluence US, and writes the result into a new Google Sheet.

**When to use**

- You need patient data plus related medical data, based on a Google Sheets test plan and/or a Confluence epic/US — the project doesn't have to be project.
- The result should land directly as rows in a new Google Sheet, not as separate JSON fixture files.
- You've provided an example FHIR resource structure (a JSON sample or a template table) — the skill then switches to FHIR mode and generates full FHIR R4 resources, writing each one as JSON text in a cell.
- You want a lighter, faster tool and don't need the full project-* infrastructure (Postman, strict validation against the project's real StructureDefinitions).

**When NOT to use (use this instead)**

- You need strict validation against project's real, project-specific FHIR definitions and the team's conventions (static-entity reuse, UI-findability, curls) → us-test-data / epic-qa-suite are more reliable for that.
- You also need a Postman collection and a test plan in one traceable package → epic-qa-suite.

**What to give the prompt as input**

- A link to a Google Sheets test plan and/or a link to/pasted text of a Confluence US or epic (at least one of the two, both is fine).
- (Optional, for FHIR mode) an example FHIR resource structure — a JSON sample or a template table of fields.
- If the source isn't obvious, the skill will ask — be ready to clarify what exactly counts as input.
- The desired number of test patients/scenarios, if that matters to you (otherwise the skill makes a reasonable assumption and states it explicitly).

**What you get as output**

- A new, separate Google Sheet with a link to it; the source test plan/Confluence page is never edited.
- Standard mode: flat columns (ID/name, date of birth, sex, diagnosis, medication, dosage, date, status, etc.).
- FHIR mode: an extra column holding the full FHIR JSON per resource as text, plus the patient's basic fields kept separate for readability.
- Coverage of the happy path plus edge cases and negative cases, where the test plan or acceptance criteria call for them.

**What to do with the output**

- Open the resulting Google Sheet and check it against the test cases/AC (valid status values, required fields filled in).
- For FHIR mode, manually check the JSON's correctness — this skill doesn't run a structural validator like validate_fixtures.py the way the project-* skills do, so that check is on the analyst.
- If the data turns out to be wrong, the same principle from section 2.3 applies: if the generator is at fault, fix the prompt/generation logic; if it's a mismatch between requirements and actual system behavior, clarify the requirements first.
- Double-check there's no accidental overlap with real personal data, even though the data is generated as synthetic.
- Use the sheet to seed a test environment or for manual QA testing.

## 4. Quick skill-selection table

| Situation | Skill to use |
| --- | --- |
| Only need FHIR fixtures for a project epic/US, nothing else | us-test-data |
| Need a full package: Postman + test plan + fixtures for a project epic (typically fixture-heavy epics) | epic-qa-suite |
| Have a Google Sheets test plan and/or Confluence US (not necessarily project), result should be a Google Sheet | test-data-generator |
| Provided a sample FHIR resource, result should be rows in a Google Sheet (not separate files) | test-data-generator (FHIR mode) |
| Need strict validation against the project's real FHIR definitions and project team conventions | us-test-data / epic-qa-suite |

## 5. Shared principles across all three skills

- Synthetic data only — never real personal data, even if the source test plan contains examples that look like real people.
- The skills never edit source documents (the test plan, the Confluence page) — they only read from them.
- If the input is incomplete or ambiguous, the skill should ask rather than guess — be ready to answer clarifying questions quickly.
- Every AI-generated FHIR output is a starting point that needs human review, even when the formal validation passes.
- If test data turns out to be wrong, what to do depends on the cause: a generator error is fixed in the skill/script itself (code); a mismatch between requirements and implementation is fixed in the requirements first. A manual patch to one fixture is a workaround, not a fix (details in section 2.3).
- Verifying the test suite is mandatory at the handoff from development to acceptance testing; doing it earlier is fine but not required (section 2.2).
- Always open and review the final output (files or spreadsheet) before passing it on to the next step of the process.

## 6. Short glossary

| Term | Explanation |
| --- | --- |
| US (User Story) | A user story in Confluence — the unit of requirements these skills work from. |
| AC (Acceptance Criteria) | The conditions under which a US is considered done. |
| FHIR | The healthcare data exchange standard (HL7 FHIR R4) that project's API is built on. |
| StructureDefinition / ValueSet | The formal definition of a FHIR resource's structure and a field's allowed codes — the "ground truth" for generating data. |
| Fixture | A ready-made test FHIR resource (JSON) used to pre-seed an environment for testing. |
| Postman collection | A set of HTTP requests for testing an API, organized into folders with automated checks (tests). |
| Static data | Organizations, doctors, medications, patients — expensive or impossible to recreate every time; created once and reused. |
| Dynamic data | Consultation notes, prescriptions, dispenses — created fresh for every test scenario. |
| Dev → acceptance handoff | The point where completed development is handed off to acceptance testing — where verifying the test suite (including the data) becomes mandatory. |

