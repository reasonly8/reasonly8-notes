```sh
# 打标签
git tag -a v1.0.0 -m "Release version 1.0.0"

# 推标签
git push origin v1.0.0

# 全推
git push origin --tags

# 看所有本地标签
git tag -l

# 删标签
git tag -d <tag-name>

# 删远程标签
git push origin :<tag-name>

# 删远程标签
git push --delete origin v1.0.0

# 看搜索远程标签
git ls-remote --tags origin
```
