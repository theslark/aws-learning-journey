# AWS Role vs Policy

## Policy
A **policy** is a document that defines **what actions are allowed or denied** on **which resources**. It's a set of rules/permissions.

Example: "Allow `s3:GetObject` on `bucket-A`."

## Role
A **role** is an **identity** that you **assume** to temporarily get permissions. It has policies attached to it.

Example: A Lambda function assumes a role that has the `S3ReadAccess` policy attached.

## Analogy

- **Policy** = the key card's **access rules** ("can enter Room 101, cannot enter Room 202")
- **Role** = the **key card itself** that someone/something picks up and uses

## In this lab

Check `data/iam-policies.json` to see the permission rules, and `data/iam-roles.json` to see which identities exist that use those rules.
