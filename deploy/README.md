# Artifact-only Tao Te Ching deployment

Keep the source checkout and all Git metadata **outside document roots**.
Build in that private checkout and publish only the contents of `dist/` to
`/mnt/www/map/polloanalytics.com/public_html/taoteching/`.
Never run `git clone`, `git init`, or `git pull` in the deployed directory.

For a reviewed release:

1. Build and inspect `dist/` in a private workspace. Check the configured Vite
   base path for the `/taoteching/` URL. Ensure the artifact has no `.git`, secret
   configuration, development settings, or source checkout files.
2. Preserve the current deployment outside all document roots. If migrating a
   legacy live checkout, archive its entire working tree (including uncommitted
   and untracked files) and preserve its `.git` directory privately before
   removing metadata from the public tree.
3. Install `apache-git-deny.conf` as root-owned
   `/etc/apache2/conf-available/taoteching-git-deny.conf`; run
   `sudo a2enconf taoteching-git-deny`, `sudo apache2ctl configtest`, and
   `sudo systemctl reload apache2`. This rule also protects nested Git metadata.
   The site's broader hidden-path policy remains independently useful.
4. Preview an artifact sync with `rsync -an --delete dist/ <deployment>/`.
   Review deletions against the private backup, then publish the reviewed
   artifacts as the deployment administrator. Do not copy the checkout itself.
   Apply the site's deployment ownership policy after publishing.
5. Verify anonymous GET requests to `/taoteching/.git/HEAD`, `.git/config`,
   `.git/refs/heads/master`, and `.git/objects/` return 403 or 404, and verify
   the page and its referenced scripts, styles, images, and data still load.
   Check both the public hostname and the origin virtual host.

## Live remediation evidence (2026-10-24)

The legacy live checkout had one commit and 16 Git objects (11 blobs). All
objects, including unreachable objects exposed by `--batch-all-objects`, were
checked for private keys, cloud/GitHub tokens, credential-bearing URLs, and
quoted password/key/token assignments. No matches were found. This targeted
static assessment cannot exclude every possible secret format or prove that
the formerly exposed history was never downloaded.

The full deployment, modified `use.txt`, untracked `dist/`, and Git history
were preserved in root-only `/var/lib/polloanalytics-git-vec-1030/`. Only `.git`
was removed from the live directory; all other deployment file hashes were
verified identical. The archived repository has no remote configured. The
private archive and assessment are not source-control artifacts.

Verified at 17:06 UTC: all four metadata paths returned **403** on both the
apex and `www` hostnames, through both public HTTPS and direct-origin requests.
All **20** deployed HTML, JavaScript, CSS, JSON and image files returned **200**
with response hashes matching the local files. Apache configuration validation
passed, and Apache remained active. The explicit Git rule above is installed
and enabled on the live server. This infrastructure/documentation change does
not publish a different application build.

Review ticket: https://quantumgazelle.com/vector/#/issue/VEC-1030
