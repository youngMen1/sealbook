# 1.Git提交规范

commit message应该如何写才更清晰明了？团队开发中有没有遇到过让人头疼的git commit？本文分享在git commit规范建设上的实践，规定了commit message的格式，并通过webhook在提交时进行监控，避免不规范的代码提交。

Git每次提交代码都需要写commit message，否则就不允许提交。一般来说，commit message应该清晰明了，说明本次提交的目的，具体做了什么操作……但是在日常开发中，大家的commit message千奇百怪，中英文混合使用、fix bug等各种笼统的message司空见怪，这就导致后续代码维护成本特别大，有时自己都不知道自己的fix bug修改的是什么问题。基于以上这些问题，我们希望通过某种方式来监控用户的git commit message，让规范更好的服务于质量，提高大家的研发效率。