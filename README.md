# youtube-dl

### 1. 준비물
  * apache2
  * python
  * youtube-dl
  * vim

### 2. 단계
  1. apache2 설치
  2. 패키지 설치
  3. youtube-dl 설치
  4. apache2 <-> cgi-bin 연동 모듈 서정
  5. cgi-bin <-> youtube-dl 연동 
  
  
### 3. 명령어 및 설정 파일 내용
```
sudo su
apt-get install -y apache2
apt-get install -y vim
apt-get install -y python

sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl
sudo chmod a+rx /usr/local/bin/youtube-dl

sudo wget https://yt-dl.org/latest/youtube-dl -O /usr/local/bin/youtube-dl
sudo chmod a+x /usr/local/bin/youtube-dl
hash -r


cd /etc/apache2/mods-enabled

ln -s ../mods-available/cgi.load cgi.load
ln -s ../mods-available/cgid.conf cgid.conf 
ln -s ../mods-available/cgid.load cgid.load

ln -s /usr/local/bin/youtube-dl /usr/lib/cgi-bin/

cd /etc/apache2/conf-available/
vim serve-cgi-bin.conf
```
* serve-cgi-bin.conf
```
<IfModule mod_alias.c>
        <IfModule mod_cgi.c>
                Define ENABLE_USR_LIB_CGI_BIN
        </IfModule>

        <IfModule mod_cgid.c>
                Define ENABLE_USR_LIB_CGI_BIN
        </IfModule>

        <IfDefine ENABLE_USR_LIB_CGI_BIN>
                ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
                <Directory "/usr/lib/cgi-bin">
                        AllowOverride None
                        AddHandler cgi-script .sh
                        Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
                        Require all granted
                </Directory>
        </IfDefine>
</IfModule>
```

```
cd /usr/lib/cgi-bin
ln -s /usr/local/bin/youtube-dl youtube-dl
vim youdl.sh
```

* youdl.sh
```
#!/bin/bash
echo "Content-type: text/html";
echo "Charset: UTF-8";
echo "";
echo "";
echo "Hello World";
#echo $1
youtube-dl -o "/var/www/html/%(title)s.%(ext)s" https://www.youtube.com/?v=${1}
```

```
chmod 755 youdl.sh

Service apache2 restart

chown -R www-data:www-data /var/www/html
```
### 4. 확인방법
  * 웹서버 URL 접속 
  ```
    http://{SERVER_URL}}/cgi-bin/youdl.sh?{{YOUTUBE_ID}}
    
  ```

  * /var/www/html 경로 밑에 동영상 다운로드 됨
  ```
    http://{SERVER_URL}}/{{VIDOE_FILE_NAME}}
    
  ```
