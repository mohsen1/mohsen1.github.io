---
layout: post
title: "How to write release notes for a breaking change"
permalink: /writing/release-notes-breaking-change/
date: 2026-11-13 00:35:00 +0100
author: Wercend
description: "A worked example for engineers explaining who needs to act, when to migrate, and how to check that the change worked."
---

Imagine finding this release note five minutes before approving a dependency upgrade.

> Version 4 introduces a new export schema. The identifier field now follows the naming convention used elsewhere in the API. We have updated the examples. The CSV column previously called `account` is now called `account_id`.

Does your weekly invoice job need a patch? The answer depends on how it reads the CSV. Someone who already knows the implementation might skim straight past that distinction.

Here is a release note for the same fictional change, with its compatibility assumptions made explicit.

> **Before upgrading to v4, check any job that reads our CSV exports by column name.** In v4 the header changes from `account` to `account_id`. If your invoice importer looks up `row["account"]`, update it to read `row["account_id"]` when you switch the export to v4. If you read columns by position, this rename requires no change because the column order and values stay the same.
>
> To check the update, export a row from v4 and import it into a test invoice. Compare the account ID on the invoice with the ID in the CSV.

## What the first draft left out

Moving the field names to the top would help. It still wouldn't tell an engineer whether a positional parser can keep working. That sentence required a fact about the export contract. In a real release, confirm it with the team that owns the exporter. If column order can change, the reassurance in the example would be wrong.

Timing matters too. A team that patches its importer on Tuesday and upgrades the exporter on Friday could break its v3 jobs for three days. “When you switch the export to v4” connects the code edit to the event that makes it necessary. Teams running both versions need a migration path covering both formats.

## Try the decision test on a draft

Hand the note to someone who maintains an integration. Ask them to underline the first sentence that tells them whether the change affects their system. Then ask which version they would test, what they would check, and what result would let them ship. Listen for answers that depend on knowledge outside the note. Those are candidates for a new sentence, a verified example, or a link to the migration guide.

For a full release with multiple changes, [The Good Docs Project's release notes guide](https://www.thegooddocsproject.dev/template/release-notes) provides a structure for organizing customer-facing notes.

This exercise comes from [Wercend](https://wercend.com/). Its public Story example lets you select a moment in a draft and follow it back to the supporting words. To try the same revision method on paper, use the [reverse outline worksheet](https://github.com/mohsen1/wercend-revision-workbook/blob/main/worksheet.md).
