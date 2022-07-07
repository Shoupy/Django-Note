# Templates and Static files
## 新增與註冊模板(Templates)
前面我們順利了建造了頁面清單和單一頁面，但基本上是單純的文字排列，也不是正規的HTML。<br>
在Django中，HTML呈現很仰賴所謂的Templates，首先先來試試看。<br>
在主目錄中，challenges資料夾中建立一個資料夾templates，然後在裡面再建立一個challenges資料夾，然後我們建造一個challenge.html<br>
亦即 ./challenges/templates/challenges/challenge.html<br>
這種建置主要是因為django對templates搜尋的設定。


如果你用了好的IDE，HTML格式一般會自動建立好。我們先隨意打一些內容
![image](https://user-images.githubusercontent.com/43126022/177176893-b50eb33a-3254-464e-ba89-6525eec789a5.png)

之後我們回到views.py，來嘗試看看把之前monthly_challenges的函式改成傳送html templates。<br>
記得從 django.template.loader 載入 render_to_string這個函式
<pre><code>from django.http import HttpResponse, HttpResponseNotFound, HttpResponseRedirect
from django.urls import reverse
from django.template.loader import render_to_string  
  
def monthly_challenges(request, month):
  try:
      challenge_text = monthly_challenges_dict[month]
      response_data = render_to_string("challenges/challenge.html")
  except:
      return HttpResponseNotFound("this month is not supported. Sorry.")
  return HttpResponse(response_data)
</code></pre>

這時候啟動去跑會發現...不能用!
原因是程式並不會搜尋challenges中的templates資料夾。
請打開子資料夾'monthly_challenges'中的settings.py
首先裡面可以看到一行
<code> BASE_DIR = Path(__file__).resolve().parent.parent </code> <br>
這表示在裡面輸入的路徑都是以我們創建這個計畫的根資料夾(monthly_challenges)為底。<br>
接下來有兩個方法可以讓django找到我們的templates
### 方法1: 直接輸入資料夾路徑
拉到settings.py中，可以看到有一區"templates"長得像這樣：
<pre><code>
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ,
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]</code></pre>
可以看到其中有個DIRS，表示搜尋的資料夾，我們在'DIRS'這行改寫為
<pre><code>'DIRS': [ BASE_DIR/'challenges'/'templates'],</code></pre>

接著去重新整理頁面，可以發現順利跑出我們剛剛設的templates。

### 方法2: 登錄apps
手動輸入路徑位置很精準，但是很麻煩，尤其大部分狀況下，某個templates應該就是給特定app用的。<br>
在templates中也可以看到'APP_DIRS'這格，是設定為True。因此第二個方法就是告訴程式，我們有某個app，可以搜尋app裡面。<br>
因此，目的就是將apps registered。同樣在settings.py中可以看到一個區塊'INSTALLED_APPS'，我們在裡面加上challenges
<pre><code>
INSTALLED_APPS = [
    'challenges',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
] </code></pre>

再去重新整理頁面(別忘了把'DIRS'先清空)，會發現這也可以。<br>

通常而言如果是在一個apps內的頁面，會建議用法二。至於有特殊需要globally被使用的頁面，也可以使用法1。<br>

這邊要闡述一下兩件事：
1. 為何templates資料夾中要再放一個challenges資料夾，才建立challenge.html呢?<br>
主要是怕搞混，因為django在搜尋templates資料夾時，會把所有templates資料夾併成一個。如果今天你有兩個以上的app，大家的templates中剛好都有challenge.html時，django是分不出來的。因此習慣上我們會在每個apps的template資料夾裡面多放上一個以自己名稱命名的資料夾。

2. render_to_string 這個函式其實是可以被用最基本的 render函式取代的，只是需要使用 <code> render(request,"challenges/challenge.html")</code>的形式，因此以後就用render吧，至於為什麼要用request，之後再慢慢解釋

## 運用模板語言 Template language 和參數插入
我們不只要呼叫模板的html架構，還要讓這個模板裡面的內容保持動態生成的。<br>
首先，可以將render新增第三個參數，裡面是一個dictionary，像這樣：
<pre><code>def monthly_challenges(request, month):
    try:
        challenge_text = monthly_challenges_dict[month]
        response_data = render(request, "challenges/challenge.html", {
            "text" : challenge_text,
        })
    except:
        return HttpResponseNotFound("this month is not supported. Sorry.")
    return HttpResponse(response_data) </code></pre>
    
這個第三個參數的dictionary，就是用來連結 html 哪些東西需要用dynamic的內容。
接著更改challenge.html的內容：
![image](https://user-images.githubusercontent.com/43126022/177370691-3501b59c-39bd-45b3-903f-db2d4e304028.png)

我們用 <code> {{ text }} </code> 來取代原先的h2區塊。 <br>
對Django來說，這個模式告訴他在讀取這個html檔案時，必須查找第三個參數所提供的dictionary中的變數來取代。<br>
Django讀取參數後會重新生成一個html頭放出來。<br>

重新runserver看看，可以發現每一個月份的任務都會自動改變了!

下一步，自己試著把每個月份的名字顯現在頁面上：
<pre><code>def monthly_challenges(request, month):
    try:
        challenge_text = monthly_challenges_dict[month]
        month_name = month.capitalize()
        response_data = render(request, "challenges/challenge.html", {
            "title": month_name,
            "text" : challenge_text,
        })
    except:
        return HttpResponseNotFound("this month is not supported. Sorry.")
    return HttpResponse(response_data)</code></pre>

HTML的部分改成:
![image](https://user-images.githubusercontent.com/43126022/177593557-91eb30bf-d7b3-4114-8ccb-648496d71d44.png)

## Filter
在上面我們傳回title的時候，在views.py中直接將其轉換成大寫。<br>
其實有另外一個方法，可以在views.py傳回小寫，然後在html中原先是 <code>{{title}}</code>的部分改成<code>{{title.capitalize()}}</code> <br>
意思是，只要在雙括弧{{ }}裡面，呼叫python的語法，django還是會能夠使用<br>

但這邊還要介紹一個django更快的方式，叫做filter。我們可以google "django template filter"，就可以到<a href = 'https://docs.djangoproject.com/en/4.0/ref/templates/builtins/'>官方頁面</a>。<br>

可以把filter想像成快速呼叫特定格式的django套件，呼叫filter要在template的html檔中把傳入的變數加上" | filter "。<br>
舉轉成大寫為例，可以將 <code>{{title}} </code> 寫成 <code> {{title|title}} </code> (剛好這個filter就叫title)
就可以看到轉換成大寫了

## For Tag
在前面的官方網站中我們可以看到另一種格式叫Tag<br>
Tag也有各式各樣的用法可以呼叫，這邊先舉For tag為例。記得之前我們在目錄區index function有點土炮的用 for loop產生月份的list嗎?<br>
現在在challenges/templates/challenges資料夾中，我們再新增一個項目叫index.html<br>

我們將index.html裡面的內容改成這樣：
![image](https://user-images.githubusercontent.com/43126022/177820392-fc94caf7-06d6-46c9-aa9c-35b8bbee0168.png)

注意 for tag 的兩個特色：
1. 以 <code> {%   %} </code>的形式
2. for tag 後面要加一個 end tag 已告知for tag的結束

接著我們將views.py中index函式改成如下：
<pre><code>
def index(request):
    months = list(monthly_challenges_dict.keys()) # ["january", "february",....]
    return render(request, "challenges/index.html", {
        "months": months,
    }) </code></pre>
    
這時候去run網頁，會發現清單創造出來了，但還無法提供連結 ...畢竟html中的a href還是空白的。

