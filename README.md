# md5index
Copyright (c) 2015 by Yura Vdovytchenko (max@itcod.com)

Copyright (c)itcod 2010-2015

local description = {

  _COPYRIGHT   = "Yura Vdovytchenko",
  
  _VERSION     = "md5index.lua 15.05.11",
  
  _URL         = "https://ihome.itcod.com/max/projects/md5index/",
  
  _DESCRIPTION = [[

   SOA ITCOD. Nginx addon for function autoindex. Add in body html:
   1. HASH code files. Support secure hash: md5 md4 sha1 sha ripemd160;
   2. Rewrite relative path body html to full URI path for files;
   3. Add extension icons for folders and files. Require icons lib. 
   Example icons lib 16x16: http://ihome.itcod.com/max/projects/libs/icons16/
   Test computation in Lua (5.1)
   Author by Yura Vdovytchenko (max@itcod.com)
  ]],
  
_LICENSE = [[

    MIT LICENSE

    Permission is hereby granted, free of charge, to any person obtaining a
    copy of this software and associated documentation files (the
    "Software"), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:

    The above copyright notice and this permission notice shall be included
    in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
    OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
  ]]
}
-----------------------------------------------------------------------------------

CONFIGURE

--path lua file: /etc/nginx/lua/md5index.lua

--Example Nginx virtual example.conf

local example = {
  _NGINX = [[

server {

    listen       80;
    server_name  dav.example.com;
    server_name_in_redirect	off;
    access_log /var/log/nginx/dav.example.com-access.log main;
    #resolver 10.255.255.1 [::1]:5353;
    charset utf-8;
    
    set $dir /opt/home;
    set $testdir $dir$uri;
    set $uri_type none;
    if (-d $testdir) { # такая папка есть
	set $uri_type dir;
	rewrite ^(.*)$ $1/;
	rewrite ^(.*)/+$ $1/;
	
    }
    if (-f $testdir) { # такой файл есть
	set $uri_type file;
    }
    if ($request_method = "MKCOL") {
	rewrite ^(.*)$ $1/;
	rewrite ^(.*)/+$ $1/;
	set $uri_type dir; #клиент webdav создает папку
    }
    if ($request_method = "PUT") {
	set $uri_type file; #передаем только файлы
    }
    if ($request_method = "POST") { 
	set $uri_type file; #постим только файлы
    }

    merge_slashes on;
    
    location / {
	#.........
	#set $file_open .htopen; #(auth-dav.lua) nil or namefiledir:hide
	client_max_body_size 0;
	autoindex on;
        root $dir;
        ####header_filter_by_lua_file /etc/nginx/lua/itcod-exchange.lua;
        
	set $md5index on; #on/off nil=off # вкл/выкл обработчик
	
	set $md5index_hash md5; #none/md5/md4/sha1/sha/ripemd160 nil=none # тип выводых хэшей
	
	set $md5index_size 50000; #kb nil=unlimit # не считать для файлов более N kb
	
	set $md5index_path on; #on/off nil=off  # заменять относительный путь ссылок на полный URI
	
	set $md5index_nonblank on; #on/off nil=off # заменить множественные пробелы одним
	
	set $md5index_type on; #on/off nil=off # добавит в строки описание типа file/directory/etc...
	
	set $md5index_ico http://ihome.itcod.com/max/projects/libs/icons16ext/; # путь к библиотека иконок
	
	set $md5index_icopref icon-; # префикс имени файла иконки
	
	#set $md5index_icosuf -icon; # суфикс имени файла иконки
	
	set $md5index_icoext .gif; # расширение файла иконки
	
        body_filter_by_lua_file /etc/nginx/lua/md5index.lua; # addon обработчик
        
    }
}
  ]]
}

------------------------

HISTORY

15.05.10 StartUp версия 

15.05.11 1.Добавлена обработка .файла доступа и блокировок .htopen 
           (qv: history auth-dav.lua 15.05.10)
         2. Добавлен ключ hide для не отображаемых папок и файлов 
            в списке autoindex
         3. Добавлено зачёркивание файлов в списке, имеющих 
            блокированный доступ (block)
