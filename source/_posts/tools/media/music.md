[toc]



## 音乐文件去重

### dupsonic

下载地址： [Releases · zas/dupsonic (github.com)](https://github.com/zas/dupsonic/releases)

```shell
# 用法
1. dupsonic scan ~/Music			# 扫描音乐库（只需做一次，之后增量更新）
2. dupsonic find-dupes
3. dupsonic find-dupes --exec "mv {} /tmp/dupes/" --keep best		# 处理重复文件，确认无误后加 --apply 执行。--keep best 会自动保留质量最高的版本，其余移走。

```

实用点：

- **增量扫描**：再次运行 `scan` 只处理新增或修改过的文件，大曲库不用重复分析[-4](https://community.metabrainz.org/t/extremely-large-music-collection-needs-advice-on-what-dedupe-program-to-use/608781/18)。
- **查看详情**：`dupsonic find-dupes --details` 会显示格式、比特率、文件大小等，方便判断保留哪个[-4](https://community.metabrainz.org/t/extremely-large-music-collection-needs-advice-on-what-dedupe-program-to-use/608781/18)。
- **JSON 输出**：支持 `--json`，方便脚本化处理[-4](https://community.metabrainz.org/t/extremely-large-music-collection-needs-advice-on-what-dedupe-program-to-use/608781/18)。