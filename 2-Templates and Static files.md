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

