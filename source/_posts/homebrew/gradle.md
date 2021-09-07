---
title: gradle 安装不通的gradle 版本
date: 2021-06-18 13:38:39
tags: brew
---

### brew 安装gradle 命令

```xml

   brew install gradle 
   这个是安装最新的gradle版本 有时候项目会冲突，所以需要安装不通的版本
   
   这个命令执行的是这个文件： https://github.com/Homebrew/homebrew-core/blob/master/Formula/gradle.rb

```
      

#### 1 下载文件的内通到本地

```xml
    class Gradle < Formula
      desc "Open-source build automation tool based on the Groovy and Kotlin DSL"
      homepage "https://www.gradle.org/"
      url "https://services.gradle.org/distributions/gradle-5.2.1-all.zip"
      sha256 "9dc729f6dbfbbc4df1692665d301e028976dacac296a126f16148941a9cf012e"
      license "Apache-2.0"
    
      livecheck do
        url "https://services.gradle.org/distributions/"
        regex(/href=.*?gradle[._-]v?(\d+(?:\.\d+)+)-all\.(?:[tz])/i)
      end
    
      bottle do
        sha256 cellar: :any_skip_relocation, arm64_big_sur: "4779bf19ae46197fa7d6e2f4f5dbbbc54ef9f4af886f7ccc4653c00532e15e9f"
        sha256 cellar: :any_skip_relocation, big_sur:       "ad28a8e146697383f34ffc5fe61506d02c6706aef34092339f7e4f36ebab4af7"
        sha256 cellar: :any_skip_relocation, catalina:      "ad28a8e146697383f34ffc5fe61506d02c6706aef34092339f7e4f36ebab4af7"
        sha256 cellar: :any_skip_relocation, mojave:        "ad28a8e146697383f34ffc5fe61506d02c6706aef34092339f7e4f36ebab4af7"
      end
    
      # gradle currently does not support Java 17
      if Hardware::CPU.arm?
        depends_on "openjdk@11"
      else
        depends_on "openjdk"
      end
    
      def install
        rm_f Dir["bin/*.bat"]
        libexec.install %w[bin docs lib src]
        env = if Hardware::CPU.arm?
          Language::Java.overridable_java_home_env("11")
        else
          Language::Java.overridable_java_home_env
        end
        (bin/"gradle").write_env_script libexec/"bin/gradle", env
      end
    
      test do
        assert_match version.to_s, shell_output("#{bin}/gradle --version")
    
        (testpath/"settings.gradle").write ""
        (testpath/"build.gradle").write <<~EOS
          println "gradle works!"
        EOS
        gradle_output = shell_output("#{bin}/gradle build --no-daemon")
        assert_includes gradle_output, "gradle works!"
      end
    end

```
> 这个文件内容复制下来 在本地目录常见一个gradle.rb 文件
  修改里面的第四行 为自己想要的版本,比如: gradle-5.2.1-all.zip
  修改里面的第五行 sha256一致，可以去这个地址看:   https://services.gradle.org/distributions  找到对应版本的sha256
  
### 2 改好后执行本地的gradle.rb 
```xml
    brew install 你的本地路径/gradle.rb
```  
>  如果提示: To install 4.4.1, first run `brew unlink gradle`.
   先执行 brew unlink gradle 然后在执行 brew install gradle.rb 
   
>  如果提示：

```xml
Updating Homebrew...
   ==> Auto-updated Homebrew!
   Updated 1 tap (homebrew/core).
   No changes to formulae.
    
 >  Warning: gradle 6.2.2 is available and more recent than version 4.4.1.
   ==> Downloading https://services.gradle.org/distributions/gradle-4.4.1-all.zip
   ==> Downloading from https://downloads.gradle-dn.com/distributions/gradle-4.4.1-
   Error: An exception occurred within a child process:
     ChecksumMismatchError: SHA256 mismatch
   Expected: 4e318d74d06aa7b998091345c397a3c7c4b291b59da31e6f9c772a596711acac
     Actual: dd9b24950dc4fca7d1ca5f1ccd57ca8c5b9eb407e3e6e0f48174fde4bb19ed06
    Archive: /Users/mayong/Library/Caches/Homebrew/downloads/abe9575f62833dd2cec95f22ff58013ed15dea648bc7fb04b884cf0a33660238--gradle-4.4.1-all.zip
   To retry an incomplete download, remove the file above.

```   
>    说明sha256不对，按照提示的真实的，再改一次，再次执行，就会成功了。

### 3 查看gradle 版本和切换
> brew info gradle  
  brew switch gradle 5.2.1
    
   



