根据用户音乐偏好推荐 20 首歌曲，覆盖「推荐歌单」。

用户输入参数: $ARGUMENTS

## 执行流程

### 第一步：校验登录状态

```bash
ncm-cli login --check
```
如果未登录，执行 `ncm-cli login --background` 并将链接给用户，等用户确认登录后再继续。

### 第二步：查询用户音乐偏好

并行执行以下命令，获取用户最新的音乐偏好数据：

1. `ncm-cli user history --limit 20 --userInput "查看最近播放记录了解偏好"` — 最近播放记录
2. `ncm-cli user favorite --userInput "查看红心歌单信息"` — 获取红心歌单信息（记录红心歌单的加密 ID，后续心动推荐需要）

提取并分析用户最近听歌的风格标签倾向（如流行、民谣、说唱、摇滚、电子、独立等），作为推荐参考。

### 第三步：确定推荐基准歌曲

**判断参数：**
- 如果 `$ARGUMENTS` 不为空（如 `《插叙人生》`），则根据参数搜索目标歌曲：
  ```bash
  ncm-cli search song --keyword "歌曲名" --userInput "搜索xxx"
  ```
  从搜索结果中选取 **visible 为 true** 的第一首歌作为推荐基准歌曲，记录其加密 ID。

- 如果 `$ARGUMENTS` 为空，则从用户最近播放记录中选取一首 **visible 为 true** 且风格具有代表性的歌曲作为推荐基准。

### 第四步：获取心动推荐

使用心动模式获取推荐歌曲：

```bash
ncm-cli recommend heartbeat --playlistId <红心歌单加密ID> --songId <基准歌曲加密ID> --count 50 --userInput "基于xxx进行心动推荐"
```

从推荐结果中筛选：
1. **visible 必须为 true**（不可播放的歌曲不要）
2. 排除基准歌曲本身
3. 排除用户已红心的歌曲（liked 为 true 的跳过，确保推荐的都是新歌）
4. 最终选取 **20 首**

如果不足 20 首，可以换一首基准歌曲再调用一次心动推荐补足。

### 第五步：查找「推荐歌单」

```bash
ncm-cli playlist created --userInput "查找推荐歌单"
```

从返回结果中找到名为「推荐歌单」的歌单，记录其加密 ID。

如果「推荐歌单」不存在，则创建：
```bash
ncm-cli playlist create --playlistName "推荐歌单" --userInput "创建推荐歌单"
```

### 第六步：清空旧歌曲

获取「推荐歌单」当前的所有歌曲：
```bash
ncm-cli playlist tracks --playlistId <推荐歌单加密ID> --limit 500 --userInput "获取推荐歌单当前歌曲"
```

如果歌单内有歌曲，批量删除：
```bash
ncm-cli playlist remove --playlistId <推荐歌单加密ID> --songIdList '["id1","id2",...]' --userInput "清空推荐歌单"
```

### 第七步：添加新推荐歌曲

将第四步筛选出的 20 首歌曲加密 ID 批量添加：
```bash
ncm-cli playlist add --playlistId <推荐歌单加密ID> --songIdList '["id1","id2",...]' --userInput "添加推荐歌曲到推荐歌单"
```

### 第八步：输出结果

以表格形式展示推荐结果，包含：
- 序号
- 歌曲名（带网易云链接，使用明文 ID：`https://music.163.com/#/song?id=<明文ID>`）
- 艺人
- 风格标签

最后附上歌单链接：`https://music.163.com/#/playlist?id=<推荐歌单明文ID>`
