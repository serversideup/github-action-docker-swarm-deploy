# Updating Release Tag
1. Publish the version as `v1.x.x` on GitHub
2. Then update the major tag locally:

```bash
git pull
git tag -fa v1 -m "Update v1 tag"
git push origin v1 --force
```