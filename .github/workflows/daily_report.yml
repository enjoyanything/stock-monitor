import akshare as ak
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from datetime import datetime, timedelta
import os
import time

# --- 配置区域 (从 GitHub Secrets 读取，稍后设置) ---
EMAIL_USER = os.getenv("EMAIL_USER")
EMAIL_PASS = os.getenv("EMAIL_PASS")
EMAIL_TO = os.getenv("EMAIL_TO")

# 股票池：代码 - 名称
STOCKS = {
    "688017": "绿的谐波",
    "603662": "柯力传感",
    "002438": "江苏神通",
    "603308": "应流股份",
    "688065": "凯赛生物",
    "600482": "中国动力",
    "688639": "华恒生物",
    "001280": "中国铀业"
}

def get_stock_news(code):
    """获取个股最近新闻"""
    try:
        # 获取新闻列表，akshare接口
        df = ak.stock_news_em(symbol=code)
        if df.empty:
            return []
        
        # 筛选过去24小时的新闻 (根据发布时间)
        # 注意：不同接口时间格式可能不同，这里做简单处理
        now = datetime.now()
        recent_news = []
        
        for _, row in df.iterrows():
            pub_time_str = str(row['发布时间'])
            # 简单清洗时间格式，适配常见格式
            try:
                if '-' in pub_time_str:
                    pub_time = datetime.strptime(pub_time_str.split('.')[0], "%Y-%m-%d %H:%M:%S")
                else:
                    continue
                
                if (now - pub_time).total_seconds() < 86400: # 24小时内
                    recent_news.append({
                        "title": row['新闻标题'],
                        "time": pub_time_str,
                        "source": row['新闻来源']
                    })
            except:
                continue
        
        return recent_news[:5] # 每个股票只取最新5条
    except Exception as e:
        return [{"title": f"获取失败: {str(e)}", "time": "-", "source": "Error"}]

def generate_report():
    report_html = f"""
    <html>
    <body style="font-family: Arial, sans-serif; line-height: 1.6; color: #333;">
        <h2 style="color: #d9534f;">🚀 【财富上台阶】每日核心资产监测报告</h2>
        <p>📅 报告时间：{datetime.now().strftime('%Y-%m-%d %H:%M')}</p>
        <p>📊 监测范围：过去24小时重磅资讯</p>
        <hr>
    """
    
    for code, name in STOCKS.items():
        news_list = get_stock_news(code)
        status_icon = "🔥" if len(news_list) > 0 else "⚪"
        
        report_html += f"""
        <div style="margin-bottom: 20px; padding: 10px; border-left: 4px solid {'#d9534f' if len(news_list)>0 else '#ccc'}; background-color: #f9f9f9;">
            <h3 style="margin: 0;">{status_icon} {name} ({code})</h3>
        """
        
        if news_list:
            report_html += "<ul>"
            for news in news_list:
                # 简单的关键词高亮逻辑
                title = news['title']
                keywords = ["定点", "订单", "投产", "涨价", "并购", "核准", "交付", "量产"]
                for kw in keywords:
                    if kw in title:
                        title = title.replace(kw, f"<b style='color:red'>{kw}</b>")
                
                report_html += f"""
                <li style="margin: 5px 0;">
                    <div style="font-weight: bold;">{title}</div>
                    <div style="font-size: 12px; color: #666;">🕒 {news['time']} | 📰 {news['source']}</div>
                </li>
                """
            report_html += "</ul>"
        else:
            report_html += "<p style='color: #999;'>今日暂无重大突发新闻。</p>"
            
        report_html += "</div>"
    
    report_html += """
        <hr>
        <p style="font-size: 12px; color: #999;">注：本报告由AI自动抓取生成，仅供参考，不构成投资建议。</p>
    </body>
    </html>
    """
    return report_html

def send_email(content):
    msg = MIMEMultipart('alternative')
    msg['Subject'] = f"【财富上台阶】每日监测 - {datetime.now().strftime('%Y-%m-%d')}"
    msg['From'] = EMAIL_USER
    msg['To'] = EMAIL_TO
    
    # 绑定HTML内容
    part = MIMEText(content, 'html', 'utf-8')
    msg.attach(part)
    
    try:
        # QQ邮箱 SMTP 服务器
        server = smtplib.SMTP_SSL("smtp.qq.com", 465)
        server.login(EMAIL_USER, EMAIL_PASS)
        server.sendmail(EMAIL_USER, EMAIL_TO, msg.as_string())
        server.quit()
        print("✅ 邮件发送成功！")
    except Exception as e:
        print(f"❌ 邮件发送失败: {e}")
        raise e

if __name__ == "__main__":
    print("开始生成报告...")
    content = generate_report()
    send_email(content)
