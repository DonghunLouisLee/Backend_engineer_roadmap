# Metadata 
author: Louis Lee   
date: 20201228  
topic: shell
# Introduction

bash_profile(zprofile)? bashrc(zshrc)? what's the difference?  

# Summary

터미널에서 'ls -a' 커맨드를 실행하면 우리는 .bash_profile 과 .bashrc 파일을 보게 된다(만약 zsh 사용자라면 .zprofile, .zshrc를 추가로 보겠지만 일단 그 부분은 뒤에 설명을 하겠습니다)

그러면 bash_profile 과 bashrc 의 차이점은 무엇일까요?

## bash_profile vs bashrc

먼저 bash can be started in interactive mode or non-interactive mode. It can also act as a login shell or a non-login shell.
기본적으로 터미널에서 bash를 실행한다면 interactive mode 이고 터미널 외에서 interactive mode 로 실행시키기 위해서는 다음과 같이 명령어를 실행하면 됩니다. 

```
bash
bash -i
bash -ic 'echo Hello!'

```
저희가 bash script를 bash 로 실행한다면 non-interactive mode 과 되는거죠. 혹은 -c 를 붙이면 됩니다. 

```
bash script.sh
bash -c 'echo Hello!'
```

자 그렇다면 bash는 언제 login shell 로 실행이 될까요?  
bash is instructed to act as a login shell when you first log in to your machine or when you start bash(1) with the --login or -l option.
login shell 은 가장 먼저 실행되서 기본적인 환경 변수 설정 등을 진행합니다. 만약 그 후에도 login shell 로 실행을 하고 싶다면 bash -l 로 실행하면 됩니다. 이 때 login shell은 bashrc 가 아닌 bash_profile 를 읽어서 실행합니다. 즉, bash_profile 은 interactive login shell에만 실행되서 딱 한번만 실행되어야 하는 커맨드를 가지고 있는 편입니다. 이에 비해 처음 로그인 후에 저희가 터미널을 통해 혹은 다른 경로를 통해 여는 bash shell은 모두 non login shell입니다. 그 중 저희가 interactive non login shell 즉, 터미널로 실행을 시킬 경우 .bashrc 에 있는 커맨드를 실행시킵니다. 즉 만약 모든 shell 마다 실행해야 하는 커맨드가 있다면 bashrc에 저장해 놓는 것이 좋습니다. 

예시를 한 번 들어보자면, PATH 는 딱 한번만 설정이 되어야 합니다. 만약 bashrc 에 PATH 를 설정하는 커맨드를 다음과 같이 저장한다면 

```
export PATH="$PATH:/addition"
```
이걸 여러번 실행하게 된다면 PATH는 /original_path:/addition:/addition:/addition 이런 식으로 무한히 늘어나게 되서 저희가 원하는 결과를 얻지 못합니다. 
그렇기 때문에 PATH같이 한번만 실행해야하는것들은 bash_profile 에 저장을 해야 합니다. 

## zsh

자 그렇다면 zsh 에서는 bash_profile 과 bashrc 를 누가 대신할까요? 바로 zprofile 과 zshrc 입니다. 이 둘이 bash_profile 과 bashrc와 역할이 정확히 일치하기 때문에
위에 설명드린대로 사용하시면 되겠습니다. 

# Notes 

https://github.com/thoughtbot/til/blob/master/bash/bash_profile_vs_bashrc.md#:~:text=bashrc%20is%20sourced%20on%20every,with%20the%20%2D%2Dlogin%20option.&text=bash_profile%20is%20great%20for%20commands%20that%20should%20run%20only%20once%20and%20.
https://apple.stackexchange.com/questions/388622/zsh-zprofile-zshrc-zlogin-what-goes-where