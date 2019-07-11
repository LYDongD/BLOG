## 最佳workflow实践

参考:

[阿里的Alone git work flow](https://mp.weixin.qq.com/s?__biz=MzAxNDU0MTE0OA==&mid=2661008528&idx=1&sn=748c3b5bdaa28c3c7b3c06614fd69d47&chksm=80feaca3b78925b5a1b9d3f075e398c567d593ebf091d9d581d8492977b347d2a7ff1e67c153&mpshare=1&scene=1&srcid=0221Z25UcgDcjve53PArNdRD#rd)

```
1 从master创建feature分支
2 在feature上开发
3 开发完成后从master创建release分支
4 从feature合并到release， 在release进行测试（测试环境）
5 测试通过后，从release合并到master，在master上发布并打tag
6 测试问题在release上修改
7 hotfix 从master拉取，修改完后合回master发布

```
