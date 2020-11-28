---
layout: post
category: webserver
title: Jekyll 설치 및 테마 설정
modified: 2017-08-17
tags: [jekyll, blog, webserver]
---

## jekyll 설치하기
jekyll을 설치해보자. jekyll을 실행하기 위해서는 4가지가 필요하다. ruby, gem, node-js, python이 그것이다. 먼저 ruby와 gem을 설치해보자.

```bash
sudo apt-get install ruby-full
wget https://rubygems.org/rubygems/rubygems-2.6.12.tgz
tar xvf rubygems-2.6.12.tgz
cd rubygems-2.6.12
sudo ruby setup.rb
```
	
이 과정이 끝나면 ruby와 gem이 설치된다. 아래와 같이 입력해 잘 설치되었는지 확인하자.

```
doocong@doocong:~/workspace/RubyGem$ ruby -v
ruby 2.3.1p112 (2016-04-26) [x86_64-linux-gnu]
doocong@doocong:~/workspace/RubyGem$ gem -v
2.6.12
```

다음은 node-js이다. node-js는 apt-get으로 다운받지 말고 .local에 한번 받아보자.(apt-get으로 받아도 무방) ubuntu를 설치했다면 항상 ~/.local/bin 은 path에 포함되어 있다. 로컬에 패키지 등을 설치해야 할때 특히 유용하다.

```bash
wget https://nodejs.org/dist/v6.11.2/node-v6.11.2-linux-x64.tar.xz
xz -d node-v6.11.2-linux-x64.tar.xz
tar xvf node-v6.11.2-linux-x64.tar
cd node-v6.11.2-linux-x64 
cp -rf * ~/.local
```
	
node-js도 잘 설치되었는지 버전을 찍어서 확인해보자.
	
```
doocong@doocong:~$ node -v
v6.11.2
```
	
python은 ubuntu 16.04를 설치했다면 기본적으로 깔려 있다. 버전만 확인하자. 확인한 이후에 ctrl-d로 나오면 된다.

```
doocong@doocong:~$ python
Python 2.7.12 (default, Nov 19 2016, 06:48:10)
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>>
```

이번에는 jekyll 설치이다. 아래 명령으로 jekyll을 설치하자. 추가로 Ruby 패키지 관리를 위한 bundler를 설치해야 한다.

```
sudo gem install jekyll bundler
```

jekyll의 버전까지 확인했다면 설치는 완벽하다.

```
doocong@doocong:~$ jekyll -v
jekyll 3.5.2
```

## jekyll 테마 설치하기

다음으로는 테마를 선택해야 한다. jekyll의 테마는 단순히 테마가 아니라 블로그의 뼈대가 된다. 테마를 교체해 주는 기능도 jekyll에 있지만, 첫 테마 설치시에는 그냥 원하는 테마를 Fork하면 된다. 나는 꽤 많은 사람들이 선택하는 [jekyll-bootstrap]( https://github.com/mmistakes/hpstr-jekyll-theme.git)을 선택했다. github로 들어가서 소스를 다운받자.

```
git clone  https://github.com/mmistakes/hpstr-jekyll-theme.git
cd jekyll-bootstrap
```

테마를 실행시키려면 필요한 Ruby 패키지를 다운받아야 한다. sudo로 root에 설치하는 것은 여러모로 후속 처리가 어렵다. 이번에는 로컬에 설치해보자. 로컬에 설치하면 이후 유지 관리가 매우 용이하다. 테마를 바꿀 일이 생기면, vendor 폴더를 지우고 다시 package를 설치하면 된다. (*)혹시나 에러가 난다면 sudo로도 설치해 보자. 문제가 해결될 수 있다.

```
bundle install --path vendor/bundle
```

이제 _config.yml을 열어서 각종 설정을 해주자. 꼭 설정해 주어야 하는 부분은 다음과 같다. destination은 Apache의 루트 폴더로 지정해 주면된다.

```yml
title:            Doocong's Journey
description:      Log of Developer Doocong
url:              http://doocong.com
destination:      /home/doocong/html
owner:
  name:           Doocong
  avatar:         avatar.jpg
  bio:            "시간이 남아돌았으면 참 좋겠다"
  email:          y103.kim@gmail.com
```

이제는 사이트를 빌드해보자. ```--watch``` 옵션을 사용하면 실시간으로 사이트의 업데이트를 볼 수 있다.

```
doocong@doocong:~/doocongs-journey$ bundler exec jekyll build --watch
Configuration file: /home/doocong/doocongs-journey/_config.yml
            Source: /home/doocong/doocongs-journey
       Destination: /home/doocong/html
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.645 seconds.
 Auto-regeneration: enabled for '/home/doocong/doocongs-journey'

```
