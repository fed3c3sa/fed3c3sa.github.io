# fed3c3sa.github.io

Static site published at **<https://fed3c3sa.github.io/>**.

It exists for one reason: a Google Cloud OAuth client that requests access to Google user data must
point at a publicly reachable **home page** and **privacy policy**, hosted on the same domain, before
its publishing status can be moved from *Testing* to *In production*. This repo is those pages.

The application they describe — *Personal Workspace Connector* — is a self-hosted, single-operator
installation of [google_workspace_mcp](https://github.com/taylorwilsdon/google_workspace_mcp) that
connects several of the operator's own Google accounts to a locally installed AI assistant. There is
no code for that application here; only its published documents.

## Pages

| URL | Purpose |
| --- | --- |
| `/` | Home page — what the application does, who runs it, link to the privacy policy |
| `/privacy.html` | Privacy policy — scopes, purposes, storage, disclosures, Limited Use commitment |
| `/terms.html` | Terms of service |
| `/it/`, `/it/privacy.html`, `/it/terms.html` | Italian translations of all three |

English is canonical at the root; Italian lives under `/it/`. The two are kept in sync — a change to
one must be mirrored in the other, since both are reachable and either could be the version a reader
lands on.

## Design constraints

These are deliberate, not omissions:

- **No JavaScript, no cookies, no analytics, no third-party requests.** A privacy policy that loads a
  tracker undermines itself, and the pages must stay readable for anyone Google sends here.
- **No build step.** Plain HTML and one stylesheet, served straight from `main`. `.nojekyll` disables
  Jekyll processing so what is committed is exactly what is served.
- **Both required URLs on one domain.** Google expects the home page and the privacy policy to share a
  domain; splitting them across hosts is the most common way this check fails.

## Editing

Edit the HTML and push to `main`; GitHub Pages redeploys within a minute or two.

When the data handling changes — a new Google API enabled, a scope added — update
`privacy.html` **and** `it/privacy.html`, bump the "last updated" date in both, and push before the
change takes effect. The commit history is the revision log both documents point readers to.

To rename the application, replace `Personal Workspace Connector` throughout and keep the name
identical to the one set in Google Cloud Console → *Google Auth Platform* → *Branding*. A mismatch
between the consent screen and these pages is exactly what a reviewer looks for:

```sh
grep -rl 'Personal Workspace Connector' --include='*.html' . \
  | xargs sed -i '' 's/Personal Workspace Connector/New Name/g'
```

## Going to production

The step-by-step Google Cloud Console procedure is in **[GO-LIVE.md](GO-LIVE.md)** (Italian).

## Licence

Site source released under the MIT licence ([LICENSE](LICENSE)). The privacy policy and terms describe
one specific personal installation — reuse the structure freely, but the statements of fact in them
are not transferable to another deployment without checking that they are still true of it.
