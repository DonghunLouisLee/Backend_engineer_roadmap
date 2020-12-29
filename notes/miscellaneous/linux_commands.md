# Metadata 
author: Louis Lee   
date: 20201229  
topic: linux basic commands
# Introduction

기본적인 linux_command 를 알아보고 예시로 알아보자. (awk는 그 자체가 일종의 프로그래밍 언어여서 아예 다른 글에서 따로 소개하도록 하겠습니다)

# Summary

## find

find is used to located files on linux

usage: find [location] -name [pattern match]
```
find . -name "*.md"
```

## grep

grep is a file pattern searcher on linux. 

usage: grep [options] [pattern match] [file]

1. grep -i: non-sensitive matching

```
ls | grep -i notes 
```

2. grep -v: invert matching

```
ls | grep -v "#" notes/miscellaneous/linux_commands.md
```

3. grep -A: print number of lines of trailing context after each match

```
ls | grep -A 1 notes
```

4. grep -B: print number of lines of trailing context before each match

```
ls | grep -B 1 notes
```

5. grep -C: print number of lines of trailing context around each match

```
ls | grep -C 1 notes
```

6. grep -n: prints out line number

```
grep -n "grep" notes/miscellaneous/linux_commands.md
```

7. grep -w: search for an entire word, not pattern 

```
grep -w "grep" notes/miscellaneous/linux_commands.md
```

8. and more

조금 특별한 사용으로는 [[::x]] 을 사용할수 있는데 x에 들어갈수 있는것들은 lower, digit, blank, space, alpha, alnum, punct, graph, upper 등이 있습니다.

```
grep "^[[:lower:]]" [file]
```

## sed

grep가 pattern matching으로 string search 하는데 쓰인다면 sed는 pattern matching으로 string modification에 쓰입니다.  
개인적으로는 scripting 할떄 sed만큼 알아두면 편리한 tool도 없다고 생각합니다. (특히 dockerfile 같은거 할때는 특히 그렇습니다)  
밑에 예시에서 -i 를 붙이지 않는 이상 실제 파일을 변경하지 않습니다. -i 를 붙여야지만 현재 output을 새로운 file로 stream에서 저장합니다.  

sed 를 기억할때 가장 중요한건 pre-flag의 역할이다. 예를 들어 sed는 기본적으로 sed '[sed_function]' 의 형식으로 이루어지는데 여기서 [sed_function] 은 [pre-flag/....] 로 이루어지게 되고 pre-flag가 sed 로 하려는 작업의 형태를 나타내게 됩니다. pre-flag로 이름을 붙인 이유는 몇몇의 작업에 한해 [sed_function] 맨 뒤에 또 flag 가 붙는데 이것들을 post-flag로 구분지어서 말하기 위함입니다.

1. viewing a range of lines of a document

```
sed -n '5,10p' [file_name]
```
for non-consecutive lines and ranges

```
sed -n -e '5,7p' -e '10,13p' [file_name]
```

2. replacing words or characters

sed replace 는 기본적으로 sed 's/[word_to_be_replaced]/[word_to_replace]/[post-flag]' [file_name]
로 이루어지는데 여기서 post-flag 는 크게 g, N, p, w가 있다. 각각의 의미는

g: global  

N: number of occurence

p: write the [word_to_replace] to standard output if replacement was made

w: append the [wored_to_replace] to file if a replacement was made 

밑의 예시들은 편의상 post-flag를 g로 고정시켜놓고 설명을 진행하겠습니다. 

```
sed 's/[word_to_be_replaced]/[word_to_replace]/g' [file_name]
```
만약 character 무시하고 replace 하려면

```
sed 's/[word_to_be_replaced]/[word_to_replace]/gi' [file_name]

```

만약 여러 blank space를 하나의 space로 바꾸고 싶다면...?   
```
sed 's/  */ /g' [file_name]
```

replacing words or characters in range

```
sed '[range_start], [range_end]s/[word_to_be_replaced]/[word_to_replace]/g' [file_name]
```

3. inserting new lines

insert는 크게 i, a 두 개의 pre-flag가 있는데 i는 목표 지점 뒤에, a는 목표 지점 전에 insert를 실행하게 됩니다. 

```
sed 'i\hello' [file_name]
```

4. deleting lines

single line  

```
sed '2d' [file_name]
```

range  
```
sed '20,35d' [file_name]
```

all the way to the end   

```
sed '3,$' [file_name]
```
 
5. modifiying lines 

특정 라인을 바꾸고 싶다면 
```
sed '3c\This is a modified line.' [file_name]
```

패턴 매칭 되는것 모두를 바꾸고 싶다면 

```
sed '/This is/c Line updated.' myfile
```


## wget

wget 을 통해 인터넷에서 파일을 다운로드할수 있다.

가장 기본적인 사용방법은  
```
wget [url]
```

다운로드할 파일 이름을 변경하고 싶다면 
```
wget -O [file_name] [url]
```

텍스트 파일에 저장된 링크에서 다운로드 할때

```
wget -i [text_file]
```

완료되지 못한 다운로드를 마저할때

```
wget -c [url]
```

백그라운드에서 다운로드 하고 싶을때

```
wget -b [url]
```

다운로드 속도를 제한하고 싶을때  
```
wget -c --limit-rate=100k [url]
```

# Notes 
https://www.tecmint.com/linux-sed-command-tips-tricks/

