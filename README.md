# 简易计算器功能测试
python app.py
# 访问 http://localhost:5000 查看效果
classDiagram
    class HS_Code {
        +string code
        +string description
        +float base_rate
        +string fta_countries
    }
    class ExchangeRate {
        +string currency_pair
        +float rate
        +timestamp updated_at
    }# 汇率更新脚本（保存为update_rates.py）
import requests
import sqlite3

def update_rates():
    res = requests.get('https://api.ecb.europa.eu/latest?base=USD')
    rates = res.json()['rates']
    
    conn = sqlite3.connect('data.db')
    c = conn.cursor()
    
    for currency, rate in rates.items():
        c.execute('''INSERT OR REPLACE INTO rates 
                     VALUES (?,?,datetime('now'))''', 
                 (f'USD_{currency}', rate))
    
    conn.commit()
    conn.close()# app.py核心代码段
@app.route('/calculate', methods=['POST'])
def calculate():
    hs_code = request.form['hs_code']
    value = float(request.form['value'])
    origin = request.form['origin']
    
    # 数据库查询
    code_data = db.execute('SELECT * FROM hs_codes WHERE code=?', 
                          (hs_code,)).fetchone()
    
    # 规则判断
    if origin in code_data['fta_countries'].split(','):
        duty = value * code_data['fta_rate']
    else:
        duty = value * code_data['base_rate']
    
    # 汇率转换
    usd_rate = get_exchange_rate('USD_CNY')
    cny_duty = duty * usd_rate
    
    return render_template('result.html', duty=cny_duty)graph LR
A[用户首次计算] --> B{分享到3个群}
B -->|是| C[解锁VIP功能]
B -->|否| D[限制每日3次]
C --> E[邀请好友得积分]
E --> F[兑换海关数据包]graph TD
A[静态网页] --> B[Flask后端]
B --> C[SQLite数据库]
C --> D[定时爬虫]
D --> E[第三方API]<!-- 免责声明模板 -->
<div class="disclaimer">
    <p>本工具计算结果仅供参考，实际清关以海关认定为准</p>
    <p>数据来源：<a href="http://www.customs.gov.cn">海关总署官网</a></p>
</div># -1
