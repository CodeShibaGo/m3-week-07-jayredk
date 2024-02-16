## 什麼是 CSRF 攻擊，該如何預防？

## 說明如何在 flask 專案中使用以下 csrf_token()語法

## ajax 需不需要使用 csrf token 進行防禦？該如何使用？

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