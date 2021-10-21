![](/static/image/微信截图_20201026110426.png)

## 多维度排序


```
        List<ShareLeaderboardVO> list = shareRecordInviteNums.stream()
                .sorted(Comparator.comparing(ShareLeaderboardVO::getClinchNum, Comparator.reverseOrder())
                        .thenComparing(ShareLeaderboardVO::getShareUid, Comparator.reverseOrder()))
                .collect(Collectors.toList());
```



