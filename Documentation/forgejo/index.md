---
title: "Connect Forgejo"
description: "Connect a Forgejo organization on Codeberg or your own server to Cratis Direct, with a personal access token or an OAuth login, so Cratis Direct can read and work on your issues, pull requests and repositories."
---

# Connect Forgejo

Cratis Direct can work with the issues, pull requests and repositories of a **Forgejo organization**, whether it lives on
[Codeberg](https://codeberg.org) or on a Forgejo server you run yourself. You connect an organization once, from
**Settings → Integrations**. Cratis Direct then acts in the organization as an account of its own, and, if you want, as one account
per agent, so every change it makes appears in Forgejo under a name you recognize.

## Before you start

- **Forgejo 15.0 or newer.** Cratis Direct reads your instance's version when you connect and stops with a clear message if it is
  older. Codeberg always meets this.
- **An organization** on the instance. A connection is one instance and one organization.
- **An account for Cratis Direct** to act as, for example `cratis-direct`, that is a member of the organization with permission to
  work in its repositories. Give it a name that tells your team who it is.
- **For your own server:** it must be reachable over HTTPS from Cratis Direct. Addresses on a private network are not reachable
  unless the operator of your Cratis Direct has allowed them.
- You must be a **maintainer** in Cratis Direct.

## Choose how Cratis Direct signs in

You choose one of two ways in the guided setup.

### A personal access token

A token works on every instance, including Codeberg.

1. Sign in to Forgejo as the account Cratis Direct will act as.
2. Open **Settings → Applications** and create a token named, for example, `Cratis Direct`.
3. Select these permissions:

   | Permission | What it is for |
   | --- | --- |
   | Repository: read and write | Reading repositories and branches; creating, merging and cleaning up pull requests |
   | Issue: read and write | Reading issues and comments; creating and updating them |
   | Organization: read and write | Reading the organization and its members; setting up the webhook |
   | User: read and write | Checking the token and setting the account's picture |

4. Copy the token and paste it into the setup. Cratis Direct checks it and tells you if a permission is missing. The token is stored
   encrypted and is never shown again.

### An OAuth login

Instead of a token you sign in to Forgejo as the account Cratis Direct will act as and approve access.

- **On Codeberg**, there is nothing to register: choose **Sign in with Forgejo** and approve. If the option is unavailable, the
  operator of your Cratis Direct has not set it up yet; use a personal access token instead.
- **On your own server**, register an OAuth application first. In Forgejo, open **Settings → Applications → Manage OAuth2
  applications**, create an application (a confidential client) and, as its redirect address, enter the one the setup shows you. It
  ends in `/integrations/forgejo/callback`. Paste the application's ID and secret into the setup, then sign in and approve.

Forgejo access tokens expire. Cratis Direct renews them for you before they lapse. If a renewal is refused, the connection shows
**Expiring** and then **Failed**, and offers **Re-authorize** so you can sign in again. A personal access token does not expire.

## Connect the organization

Open **Settings → Integrations → Connect → Forgejo**. Each step is saved, so you can leave and come back where you stopped.

1. **Instance and organization.** Choose Codeberg, or enter your server's address, for example `https://git.example.org`, and
   enter the organization's name. Organizations do not nest, so a name is enough.
2. **Sign in.** Choose a personal access token or an OAuth login and complete it as described above.
3. **Default identity.** This is the account every agent can act as. It comes from the token or login you just gave.
4. **Agent accounts (optional).** By default, every agent acts as the default account. If you want each agent to appear under its
   own name, either:
   - give an **administrator's** token, and Cratis Direct creates one account per agent, named `cratis-direct-<agent>` and shown as
     `Cratis Direct – <Agent>`, each with its own picture; or
   - **adopt** accounts you already created, by giving each account's token.

   Skip this step to keep a single account.
5. **Webhooks.** Cratis Direct registers one webhook on the organization (or on each repository, if the account cannot create an
   organization webhook) for issues, comments, pull requests, reviews and pushes, so changes reach Cratis Direct without polling.
   Cratis Direct signs and verifies every delivery with a secret it generates.
6. **Verify.** The agent grid shows each agent as **Ready**, or says what is missing and offers the action that fixes it. Agents
   without their own account are shown acting as the default account.

Running the setup again is safe: whatever is healthy is left alone and whatever is missing is made again.

## What Cratis Direct does in Forgejo

- **Issues.** Reads and mirrors issues, comments, labels, assignees and state, and publishes changes made in Cratis Direct. Issues
  are identified by the number they have in their repository.
- **Pull requests.** Lists, reads, creates and merges pull requests, using a merge style your repository allows, and deletes the
  branch afterwards. Bringing a branch up to date is offered only if your Forgejo offers it; otherwise Cratis Direct says so.
- **Repositories.** Lists repositories and branches and clones them, on Codeberg and on your own server alike.

Not included: releases, builds and CI status (Forgejo Actions), and packages.

## Security

- Tokens, OAuth secrets and webhook secrets are stored encrypted and never appear in logs or messages.
- Only a maintainer can connect an organization, and the sign-in flow is short-lived and tied to the person who started it.
- Cratis Direct only reaches HTTPS addresses, and never your internal network unless your operator has allowed a specific host.
- Cratis Direct recognizes its own accounts and ignores their changes when they come back through webhooks.

## Troubleshooting

| Message | What to do |
| --- | --- |
| The instance is older than 15.0 | Upgrade Forgejo to 15.0 or newer, then connect again. |
| A permission is missing from the token | Create a new token with the permissions in the table above. |
| The address is refused | Use an `https` address without a username or password, on a host reachable from the internet, or ask your operator to allow your private host. |
| The connection shows Expiring or Failed | Choose **Re-authorize** and sign in again. |
| An agent's picture was not set | The agent still works; Cratis Direct tries again later. |
