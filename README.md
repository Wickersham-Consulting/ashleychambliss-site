# ashleychambliss-site

Static rebuild of [ashleychambliss.com](https://ashleychambliss.com/) — a single-page artist site for singer/songwriter Ashley Chambliss.

The live site at `ashleychambliss.com` is currently on managed WordPress (Divi). This is a plain HTML/CSS port deployed to AWS S3 + CloudFront.

## Staging URL

https://ashley.wickershamconsulting.com/

The eventual cutover will move `ashleychambliss.com` itself to AWS; this staging subdomain disappears at that point.

## Layout

```
site/                     ← deployed verbatim to S3
  index.html
  style.css
  img/
  video/
.github/workflows/
  deploy.yml              ← syncs site/ on every push to main
```

## Local preview

```sh
python3 -m http.server 4848 --directory site
# → http://localhost:4848/
```

## Deploy

Pushes to `main` trigger `.github/workflows/deploy.yml`, which:
1. Assumes a least-privilege IAM role via GitHub OIDC.
2. `aws s3 sync site/` to the bucket (with `--delete`).
3. Invalidates the CloudFront cache (`/*`).

Manual deploy from a local checkout (skips CI):

```sh
aws --profile wc-cwickersham s3 sync site/ s3://wc-site-ashleychambliss-010119294271/ --delete
aws --profile wc-cwickersham cloudfront create-invalidation \
  --distribution-id ED1NLP8JW0UKZ --paths '/*'
```

## Infra

S3 bucket, CloudFront distribution, ACM cert, and Route 53 zone are managed in [Wickersham-Consulting/infra](https://github.com/Wickersham-Consulting/infra) via the `static-site` OpenTofu module.
