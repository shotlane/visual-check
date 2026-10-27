# Shotlane Visual Check

Checks every preview deploy on phone, tablet and desktop, and posts the screenshots plus any problems on the pull request:

- content wider than the screen (sideways scrolling), with the element that causes it
- a missing viewport tag
- uncaught JavaScript errors and console errors
- failed requests and broken images
- text squeezed into a narrow column (words breaking letter by letter), text cut off by its box, and overlapping text
- text below WCAG AA contrast
- tap targets under 24×24px on phones (WCAG 2.5.8)

It works with Vercel, Netlify, Cloudflare Pages and anything else that reports GitHub deployment statuses. One comment per PR, updated on every deploy.

## Setup

1. Get an API key at [shotlane.dev](https://shotlane.dev) (100 free credits a month) and add it as the repository secret `SHOTLANE_API_KEY`.
2. Add `.github/workflows/shotlane.yml`:

```yaml
name: Shotlane visual check
on: deployment_status
permissions:
  contents: read
  pull-requests: write
jobs:
  check:
    if: github.event.deployment_status.state == 'success' && !contains(github.event.deployment.environment, 'roduction')
    runs-on: ubuntu-latest
    steps:
      - uses: shotlane/visual-check@v1
        with:
          api-key: ${{ secrets.SHOTLANE_API_KEY }}
          pages: "/ /pricing"
```

Each page costs 1 credit per device: two pages on three devices is 6 credits per deploy.

### Vercel Deployment Protection

If your previews need a login, create a bypass secret (Project Settings → Deployment Protection → Protection Bypass for Automation), store it as `VERCEL_AUTOMATION_BYPASS_SECRET` and add:

```yaml
          vercel-bypass-secret: ${{ secrets.VERCEL_AUTOMATION_BYPASS_SECRET }}
```

### A fixed URL instead of deployments

```yaml
on: pull_request
# ...
      - uses: shotlane/visual-check@v1
        with:
          api-key: ${{ secrets.SHOTLANE_API_KEY }}
          url: https://staging.example.com
```

## Inputs

| Input | Default | |
|---|---|---|
| `api-key` | (required) | Shotlane API key |
| `url` | the deployment URL | Base URL to check |
| `pages` | `/` | Space-separated paths |
| `devices` | `iphone,ipad,desktop` | Any of iphone, iphone_se, pixel, galaxy, ipad, laptop, desktop, desktop_hd |
| `vercel-bypass-secret` | | For protected Vercel previews |
| `comment` | `true` | Post and update one PR comment (results always go to the job summary) |
| `fail-on-issues` | `false` | Fail the step when problems are found |
| `github-token` | `github.token` | Needs `pull-requests: write` |

Output: `issues`, the number of problems found.

Screenshots are hosted at unguessable links for 30 days, then deleted. API docs: [shotlane.dev/docs/devices](https://shotlane.dev/docs/devices#what-it-checks).

## Legal

[Legal Notice](https://shotlane.dev/legal-notice) - [Terms](https://shotlane.dev/terms) - [Privacy](https://shotlane.dev/privacy)

## License

MIT
