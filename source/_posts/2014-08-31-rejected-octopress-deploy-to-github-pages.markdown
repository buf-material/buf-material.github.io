---
layout: post
title: "OctopressのコンテンツデータをGitHub Pagesへdeployしようとしたらrejectedで失敗する件"
date: 2014-08-31 21:17:02 +0900
comments: true
categories: 
  - Octopress
---

[Octopress](http://octopress.org/)を[GitHub pages](https://pages.github.com/)でホスティングさせているのですが、**GitHubレポジトリへのdeployタスク**が`rejected`となって失敗してしまうことがあります。

```
## Pushing generated _deploy website
Enter passphrase for key '/home/maehachi08/.ssh/id_rsa':
To git@github.com:buf-material/buf-material.github.io.git
 ! [rejected]        master -> master (non-fast-forward)
error: failed to push some refs to 'git@github.com:buf-material/buf-material.github.io.git'
To prevent you from losing history, non-fast-forward updates were rejected
Merge the remote changes before pushing again.  See the 'Note about
fast-forwards' section of 'git push --help' for details.

## Github Pages deploy complete
cd -

```

## 暫定対処
他にも解決方法があるかもしれませんので暫定と思っています。

Octopressの`Rakefile`に記述されている**deployタスク内で実行されているpushタスクをforceオプションで実行**します。

`git push`コマンドのremoteレポジトリ名の頭に`+`を付加することでforceオプションと同じ効果を付加できるようです。

```diff diff of Rakefile
--- a/Rakefile
+++ b/Rakefile
@@ -265,7 +265,7 @@ multitask :push do
     puts "\n## Committing: #{message}"
     system "git commit -m \"#{message}\""
     puts "\n## Pushing generated #{deploy_dir} website"
-    Bundler.with_clean_env { system "git push origin #{deploy_branch}" }
+    Bundler.with_clean_env { system "git push origin +#{deploy_branch}" }
     puts "\n## Github Pages deploy complete"
   end
 end
```

