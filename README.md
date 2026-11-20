# Project 07 --- CRM Data Hygiene & Deduplication

An n8n workflow that retrieves HubSpot contacts, standardizes contact
fields, validates records, identifies potential duplicates, and prepares
a clean CSV of valid, non-duplicate contacts for downstream GTM
workflows.

<img width="934" height="344" alt="screenshot" src="https://github.com/user-attachments/assets/bf8aed3b-a46c-4988-a8ac-50d91b5830d0" />


## Problem

CRM data can contain incomplete fields, inconsistent formatting, and
duplicate contact records. These issues can reduce the reliability of
segmentation, prospecting, reporting, and sales handoffs.

This project demonstrates a repeatable data-quality workflow that checks
contact records before they are used in downstream GTM processes.

## Workflow overview

1.  **Retrieve contacts** --- Fetch contacts from HubSpot using the
    HubSpot node.
2.  **Normalize fields** --- Standardize names, email addresses, phone
    numbers, and company names.
3.  **Validate records** --- Check required fields and basic email and
    phone validity.
4.  **Detect exact duplicates** --- Group contacts by matching email
    address or normalized phone number.
5.  **Detect fuzzy matches** --- Compare contact names and company names
    using Levenshtein similarity.
6.  **Consolidate match signals** --- Combine exact and fuzzy match
    evidence into duplicate pairs.
7.  **Prepare output records** --- Select the fields needed for the
    final CSV.
8.  **Export valid unique contacts** --- Flatten the unique-contact
    dataset and convert it to CSV.

## Validation rules

-   First name is required.
-   Last name is required.
-   Email is required and must pass a basic email-format check.
-   Phone number is optional; when provided, it must contain between 7
    and 15 digits.

These are practical baseline checks, not full verification of identity,
deliverability, or phone-number ownership.

## Duplicate-detection logic

### Exact matching

-   Email addresses are trimmed and compared in lowercase.
-   Phone numbers are compared after removing non-digit characters.
-   For this test dataset, a leading Indian country code (`91`) is
    removed from 12-digit phone numbers so that a number with `+91` can
    match its 10-digit form.

### Fuzzy matching

The workflow compares every pair of contacts using Levenshtein edit
distance.

A pair is flagged as a fuzzy candidate when both conditions are met:

-   Name similarity is at least **85%**.
-   Company similarity is at least **75%**.

These thresholds are heuristic review signals, not probabilities or
proof that two records belong to the same person. Fuzzy matches should
be reviewed before any CRM merge or deletion.

## Test results

Tested with a sample HubSpot dataset of 26 contacts.

  Measure                                      Result
  ------------------------------------------ --------
  Contacts processed                               26
  Valid contacts                                   26
  Invalid contacts                                  0
  Contacts flagged as duplicates                    4
  Duplicate pairs                                   2
  Valid, non-duplicate contacts exported           22
  Contact pairs compared by fuzzy matching        325

The two duplicate pairs were identified by phone matching and
independently corroborated by fuzzy name-and-company matching.

Results describe this test dataset and are not a guarantee of the
workflow's performance on other CRM data.

## Output

The workflow prepares three datasets internally:

-   `unique_contacts.csv` --- valid contacts not flagged as duplicates.
-   `duplicate_contacts.csv` --- contacts flagged by duplicate
    detection.
-   `invalid_contacts.csv` --- contacts that fail validation.

The current export path converts the unique-contact dataset to CSV. The
tested output contained **22 records** and these columns:

`hubspotId`, `firstName`, `lastName`, `email`, `phone`, `company`,
`jobTitle`, `lifecycleStage`

## Requirements

-   n8n (tested with version 2.47.4)
-   A HubSpot account
-   A HubSpot credential configured in n8n with permission to read the
    required contact properties

The workflow export does not include a usable credential. After
importing it, configure your own HubSpot credential in n8n and select it
in the **Get many contacts** node.

## Import and run

1.  Import the workflow JSON into your n8n instance.
2.  Open the **Get many contacts** node.
3.  Select your own HubSpot credential.
4.  Confirm that the credential has read access to the contact data used
    by the workflow.
5.  Review the selected contact properties and adjust them to match your
    HubSpot property setup.
6.  Execute the workflow.
7.  Inspect the output of the **Convert to File** node and download the
    generated CSV.

## Security and privacy

-   Never commit HubSpot access tokens, API keys, passwords, or
    credential exports.
-   Do not commit real contact exports containing names, email
    addresses, phone numbers, or CRM record IDs to a public repository.
-   Use synthetic or anonymized sample data when demonstrating the
    output.
-   Keep credentials in n8n's credential store and use your own
    credentials after importing the workflow.
-   Review fuzzy-match candidates before making changes to CRM records.

## Scope and limitations

-   The workflow reads contacts; it does not merge, update, or delete
    HubSpot records.
-   Email validation checks basic syntax only; it does not confirm that
    an inbox exists or can receive mail.
-   Phone validation checks digit count only; it does not verify number
    assignment or reachability.
-   The Indian `91` phone-prefix normalization is specific to the test
    dataset and should be adapted for international datasets.
-   Fuzzy matching uses all-pairs comparison, which is suitable for this
    small test dataset but may need blocking or indexing for larger
    contact databases.
-   Similarity thresholds may produce false positives and false
    negatives. Human review is recommended before acting on matches.

## Tech stack

-   **n8n** --- workflow orchestration and automation
-   **HubSpot** --- CRM contact source
-   **JavaScript** --- normalization, validation, exact matching, fuzzy
    matching, and consolidation
-   **CSV** --- portable output for downstream workflows

------------------------------------------------------------------------

Built as a hands-on GTM Engineering project focused on CRM data quality
and workflow automation.
