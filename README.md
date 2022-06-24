# Django-Note

## 1. 新增 Django Project
如果想要新增一個project mypage
在command line中移到想要建立project的基底資料夾，然後輸入 django-admin startproject mypage

<pre><code> django-admin startproject mypage </code></pre>

Django會自動建立一個資料夾mypage，並創建相關的內容

## 2. 啟動local server
在基底資料夾中，可以看到第二層mypage資料夾，以及manage.py，這是django自動建立的系統操作檔，只要使用，不需去編碼他。
在Command line中進入基底資料夾(有manage.py的那層)，然後輸入
<pre><code>
> python manage.py runserver
</code></pre>

之後會顯示如下結果：
<pre><code>
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
June 25, 2022 - 07:37:53
Django version 3.2.5, using settings 'mypage.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CTRL-BREAK.
</code></pre>

之後在瀏覽器上進入http://127.0.0.1:8000/ 應可順利看到Django範本葉面

如要結束server running。可以在Command line頁面中按下 Ctrl + c

## 3. 新增App
Django 中每一個功能/頁面不是直接寫出來，而是以模組的形式完成，然後再呼叫模組。
通常一個特定的項目就會寫成一個 app. 
現在來新增第一個app，在command line中進入基底資料夾(有manage.py)，輸入：

<pre><code> 
> python manage.py startapp challenges 
</code></pre>

這樣一來我們就建立了一個叫做challenges的app

