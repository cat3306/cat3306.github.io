# iterm2和zsh


<!--more-->

1. 下载 [iTerm2](https://iterm2.com/)

2. zsh 系统自带，但不是最新，安装最新zsh

```bash
➜  ~ zsh --version
zsh 5.8 (x86_64-apple-darwin18.7.0)
➜ brew install zsh
```

1. 安装oh-my-zsh

   ```bash
   git clone git@github.com:toxicwebdev/robbyrussell-oh-my-zsh.git ~/.oh-my-zsh
   # or
   git clone git@github.com:genDopamine/ohmyzsh.git ~/.oh-my-zsh
   ```

2. 配置oh-my-zsh

   ```bash
   cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
   ```

3. 兼容bash环境变量

   ```bash
   echo "source ~/.bash_profile" >> ~/.zshrc #刷新bash的环境变量 ubuntu下不行可拷贝
   ```

4. 修改shell

   ```bash
   chsh -s /bin/zsh
   ```

5. 更换主题
   [主题图鉴](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)

   ```bash
   vim ~/.zshrc //修改ZSH_THEME="avit"
   ```

   ![](/img/iterm2/1.png)

   挑几个自己喜欢的主题

   - avit

   ![](/img/iterm2/2.png)

   - robbyrussell

   ![](/img/iterm2/3.png)

6. [插件](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins) oh-my-zsh 可以绑定插件，可以变的更加漂亮。

   - 命令高亮

     1. ```bash
        brew install zsh-syntax-highlighting
        ```

     2. ```bash
        git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
        ```

     3. ```bash
        vim ~/.zshrc #在 plugins=(...) 增加zsh-syntax-highlighting
        ```

        ![](/img/iterm2/4.png)

   - 自动补全命令

     1. ```bash
        git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
        ```

     2. ```bash
        vim ~/.zshrc #在 plugins=(...) 增加zsh-autosuggestions
        ```

        ![](/img/iterm2/5.png)

   - 官方插件非常多 [plugins](https://github.com/ohmyzsh/ohmyzsh/wiki/Plugins) 激活方式

     ```bash
      vim ~/.zshrc #在 plugins=(...) 增加插件
     ```

     注：
     如果插件下不下来(墙的问题)可copy https://gitee.com/Joker_null/plugins/tree/master/zsh 下的文件夹到 

     ```bash
     cp -r ./plugins/zsh/zsh-syntax-highlighting ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
     
     cp -r ./plugins/zsh/zsh-autosuggestions ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
     
      vim ~/.zshrc #在 plugins=(...) 增加zsh-syntax-highlighting
      vim ~/.zshrc #在 plugins=(...) 增加zsh-autosuggestions
     ```


- iterm2配置rz,sz

  1. 安装lrzsz

     ```
     brew install lrzsz
     ```

  2. 复制两个shell脚本到 `/usr/local/bin` 下

     ```
     https://gitee.com/Joker_null/plugins/blob/master/zsh/iterm2-recv-zmodem.sh
     https://gitee.com/Joker_null/plugins/blob/master/zsh/iterm2-send-zmodem.sh
     ```

  3. 打开iterm2 Preferences -> Profiles -> Default -> Advanced 的 tab 页 -> Triggers - Edit

     ```
     Regular expression: rz waiting to receive.\*\*B0100
                 Action: Run Silent Coprocess
             Parameters: /usr/local/bin/iterm2-send-zmodem.sh
                Instant: checked
     
     Regular expression: \*\*B00000000000000
                 Action: Run Silent Coprocess
             Parameters: /usr/local/bin/iterm2-recv-zmodem.sh
                Instant: checked
     ```

  ![](/img/iterm2/6.png)

  ![](/img/iterm2/7.png)


