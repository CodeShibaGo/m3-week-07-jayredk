## 什麼是 CSRF 攻擊，該如何預防？
Cross Site Request Forgery(CSRF)，中文翻成跨站請求僞造
簡單來說就是駭客的網站僞裝成使用者，並向使用者先前登入過的網站發送請求，產生非使用者預期內的行為，藉此盜取使用者的金錢、帳號等
> NOTE：使用者指的是造訪駭客網站的人、CSRF 攻擊的受害者

### 作為一個網站的開發者，該如何預防 CSRF 攻擊？

### Step 1：了解如何預防 XSS 攻擊
在預防 CSRF 攻擊前，首要的是了解如何預防 XSS 攻擊，

因為 XSS 攻擊可以破解一切預防 CSRF 攻擊的手段，所以可以先把這篇內容讀過一遍：[Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)

### Step 2：檢查網站使用的 framework 是否有內建防禦 CSRF 攻擊的功能
若有的話，大膽的用下去！

像是 .NET 本身就有內建防禦 CSRF 攻擊的機制

沒有的話可以跳到下一步

### Step 3：網站是否為 stateful?
是的話就使用 Synchronizer Token Pattern 吧！

> Note：如果不是可以直接跳到下一步

#### Synchronizer Token Pattern 是什麼？

簡單來說，就是在每次使用者發出請求或是建立 session 時，在 Server 產出一個 CSRF token 並送給使用者，藉此 token 來判斷使用者的請求是否有效

#### CSRF token 的目的與必要條件

CSRF token 的目的是讓駭客僞裝使用者發送的請求無效，因為只要駭客拿不到 token，發再多請求都是無效的

因此，CSRF token 必須符合三個條件：
1. 在一個 user session 中是唯一的
2. 只有伺服器知道（產生 CSRF token 的一方）
3. 不可預測（隨機算數產生），可以參考[連結](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#rule---use-cryptographically-secure-pseudo-random-number-generators-csprng)的方式

#### CSRF token 的期限
CSRF token 的期限分成兩種：
1. 在 session 結束前都有效（per-session token）
2. 單一請求（per-request token）

一般來說，單一請求的 token 會更安全，因為駭客必須在單一 request 拿到使用的 token 是比較困難的。

相對於在 session 結束前的 token 擁有更少的時間

但是單一請求的 token 有一個缺點就是會阻礙使用者與網站互動的有效性

例如瀏覽器的上一頁功能，當使用者回到上一頁時，先前頁面的 token 就會失效。

反之，使用 per-session token 就沒有這個問題。

#### CSRF token 的傳遞方式

CSRF token 在伺服器傳遞給使用者時，可以透過 HTML、JSON 的形式傳遞

使用者要回傳 CSRF token 至伺服器時，則有非常多種方式

1. 塞在表單中的隱藏欄位，透過表單送出的方式回傳
2. 塞在 AJAX 請求中的自訂 headers 欄位回傳
3. 塞在 JSON 資料中的其中一個欄位回傳

要特別注意的是 CSRF token 不能塞在 cookie 中進行傳遞，

且必須確保 CSRF token 不會出現在伺服器產生的 logs、或是 URL 中（祕密不能被泄漏出去）


> 後記：發現這個坑蠻大的，只寫了基本的 Synchronizer Token Pattern，後續再寫一篇完整的文章


Reference：
- [Cross Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf)
- [Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#cross-site-request-forgery-prevention-cheat-sheet)
- [零基礎資安系列（一）-認識 CSRF（Cross Site Request Forgery）](https://tech-blog.cymetrics.io/posts/jo/zerobased-cross-site-request-forgery/)
- [讓我們來談談 CSRF](https://blog.techbridge.cc/2017/02/25/csrf-introduction/)

## 說明如何在 flask 專案中使用以下 csrf_token()語法
> Note：觀看以下步驟前請確保已安裝 Flask-WTF
>
> 若尚未安裝可輸入 `pip install -U Flask-WTF`

這裡分為兩種情況：有無使用 FlaskForm

如果有，可以直接跳至 Step 2，並專注於 SECRET_KEY 的部分即可

若沒有則從 Step 1 開始

### Step 1：匯入 CSRFProtect 並套用至 app
```python
from flask import Flask,
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
csrf = CSRFProtect(app)
```

### Step 2：設定 SECRET_KEY
```python
from flask import Flask,
from flask_wtf.csrf import CSRFProtect
import secrets  # 第一步：匯入 secrets 

secret_key = secrets.token_hex(32)  # 第二步：隨機產生一組 16 進制 32 字元的字串

app = Flask(__name__)
app.config['SECRET_KEY'] = secret_key  # 第三步：設定 SECRET_KEY
csrf = CSRFProtect(app)
```

### Step 3：在 HTML forms 中使用

#### 情況 1：有使用 FlaskForm
> Note: LoginForm 為使用了 FlaskForm 的 class

Step 1: 創建 LoginForm 後作為 form 參數傳遞至樣板
```python
from app.forms import LoginForm

@app.route('/login', methods=['GET'])
def login():
    form = LoginForm()
    
    return render_template('login.html', title='Sign In', form=form)
```

Step 2:透過 `form.csrf_token` 即可渲染出 csrf token
```html
<form method="post">
    {{ form.csrf_token }}
</form>
```
> Note： 還有一種寫法是 `form.hidden_tag()`，用於一次渲染多個 hidded field
>
> Reference:[Creating Forms - flask-wtf](https://flask-wtf.readthedocs.io/en/0.15.x/quickstart/#creating-forms)

#### 情況 2：沒有使用 FlaskForm

Step 1:在 view function 先匯入 `generate_csrf`，並作為 csrf_token 參數傳送至樣板

```python
from flask_wtf.csrf import generate_csrf

@app.route('/login', methods=['GET'])
def login():
    return render_template('login.html', title='Sign In', csrf_token=generate_csrf)
```

Step 2:在樣版中直接透過 `csrf_token()` 使用
```html
<form method="post">
    <input type="hidden" name="csrf_token" value="{{ csrf_token() }}"/>
</form>
```


Reference：[CSRF Protection - flask-wtf](https://flask-wtf.readthedocs.io/en/0.15.x/csrf/)

## ajax 需不需要使用 csrf token 進行防禦？該如何使用？

> Note：在開始前請確保有下列程式碼，否則會出錯

```python
from flask import Flask
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
csrf = CSRFProtect(app)
```

### 如何在使用者端（client side）透過 AJAX 夾帶 CSRF token？
> Note：這部分文件寫的很爛，沒頭沒尾看不太懂
>
> 可以理解成頁面中可能有某些操作是 API call，例如上傳檔案、獲取資料等等
>
> 那這個 API call 也要防止 csrf token 的攻擊
>
> 因此伺服器端（flask application）可以在返回頁面的時候設定好哪個按鈕會執行 API call，並提前在 headers 中塞入 csrf token 

#### 使用 axios
```html
<script type="text/javascript">
    axios.defaults.headers.common["X-CSRFToken"] = "{{ csrf_token() }}";
</script>
```

#### 使用 fetch
```html
<script type="text/javascript">
  fetch('/your-api-endpoint', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': "{{ csrf_token() }}"  // Include the CSRF token in the headers
    },
    body: JSON.stringify({ data: 'your_data' })
  })
</script>
```

## 如何使用 Virtualenv 建立環境？
```bash
python -m venv .venv
```
## 調教 VS Code 讓 VirtualEnv 環境更好用

### Step 1：確認 VS code 自動啟用虛擬環境的設定是否開啟
預設 VS code 應該會設定：
```
"python.terminal.activateEnvironment": true
```

若沒有就要打開 `settings.json` 自行加入

記得要加在 User 的 `settings.json` 這樣才會套用到所有專案

### Step 2：確保虛擬環境使用`.venv`進行儲存
到這步就完成了

接下來只要終端機開啟，VS code 就會自動啟用虛擬環境

## 如何測試虛擬環境是否有載入成功？
看看 prompt 的左側是否有出現 (venv)
> Note：若有改過終端機介面的則不在此限

例如：
載入前
```bash
$
```

載入後
```bash
(venv) $
```

## 如何判斷套件是否安裝成功？

### 方法 1：pip show <套件名稱>
```bash
pip show flask
```

若有找到就會顯示相關資訊：如版本、安裝位置等等

反之，未找到則會出現訊息：
```
WARNING: Package(s) not found: <套件名稱>
```

### 方法 2: pip list
直接列出所有安裝套件，按字母順序看看套件名稱有沒有出現

找得到就是有安裝，反之就是沒有

---
## 以下問題為自行添加

## 如何啟用虛擬環境
### Mac or Linux
```bash
source venv/bin/activate
```

### Windows
```powershell
venv\Scripts\activate
```

## 如何取消虛擬環境？
```bash
deactivate
```