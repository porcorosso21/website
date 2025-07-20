# Wiley TDM Client

使用 Wiley 官方 Text and Data Mining Python 工具下載論文

可至官方頁面取得API金鑰 <https://onlinelibrary.wiley.com/library-info/resources/text-and-datamining>

Wiley TDM Client Github 頁面 <https://github.com/WileyLabs/tdm-client?tab=readme-ov-file#wiley-tdm-client>

```python
import requests
import json
import re
import os
import csv
import sys
from datetime import datetime
from time import sleep
from wiley_tdm import TDMClient

# 定義參數
class ClassConfig:
    apikey = 'API KEY'  # Wiley TDM API 金鑰
    issn = '1600-0579'  # ISSN 號碼
    sdate = '2025-01-01'  # 起始日期
    edate = '2025-12-31'  # 結束日期
    url = f"https://api.crossref.org/v1/journals/{issn}/works?rows=2&select=container-title,DOI,title,author,created&filter=from-pub-date:{sdate},until-pub-date:{edate}"  # 替換成實際的 API URL
    download_folder = 'downloads'  # 下載資料夾
    debug = False  # 是否開啟除錯模式

class ClassJournal:
    doi = ''
    created_date = ''
    title = ''
    author = ''
    download_result = ''
    download_file_org_name = ''
    download_file_new_name = ''

# 初始化
config = ClassConfig()
# 建立下載資料夾
if not os.path.exists(config.download_folder):
    os.makedirs(config.download_folder)
if config.debug:
    print("DEBUG模式...")


# 取得所有文章
print(" ")
print("取得所有文章...")
print(f"URL: {config.url}")
# 發送 GET 請求
try:
    response = requests.get(config.url)
    result = response.json()
except requests.exceptions.RequestException as e:
        print(f"請求錯誤: {e}")
        sys.exit()
except json.JSONDecodeError as e:
        print(f"JSON 解析錯誤: {e}")
        sys.exit()
items = result['message']['items']
journals = []
for item in items:
    journal = ClassJournal()
    journal.container_title = item.get('container-title', [])
    journal.container_title = journal.container_title[0] if journal.container_title else 'NULL'
    journal.doi = item.get('DOI', 'NULL')
    journal.created_date = str(item['created']['date-time']).split('T')[0]
    journal.created_date =  datetime.strptime(journal.created_date, "%Y-%m-%d").strftime("%Y_%m")  # 格式化日期為 YYYYMMDD
    journal.title = item.get('title', [])
    journal.title = journal.title[0] if journal.title else 'NULL'
    journal.author = item.get('author', [])
    journal.author = journal.author[0]['given'] + ' ' + journal.author[0]['family'] if journal.author else 'NULL'
    journal.download_file_org_name = str(journal.doi).replace('/', '-') + '.pdf'
    journal.download_file_new_name = str(journal.created_date) + '_' + journal.author + '.pdf'

    journals.append(journal)
    
    if config.debug:
        print("--------------------------")
        print(f"期刊: {journal.container_title}")
        print(f"DOI: {journal.doi}")
        print(f"日期: {journal.created_date}")
        print(f"標題: {journal.title}")
        print(f"作者: {journal.author}")
        print(f"下載檔名: {journal.download_file_org_name}")
        print(f"重新命名檔名: {journal.download_file_new_name}")

# 檢查是否有找到文章
if len(journals) == 0:
    print("沒有找到任何文章，請檢查 ISSN、起始日期和結束日期是否正確。")
    sys.exit()

# debug模式在此暫停
if config.debug:
    sys.exit()

# 刪除資料夾內所有檔案
print(" ")
print("刪除資料夾內所有檔案...")
for item in os.listdir(config.download_folder):
        item_path = os.path.join(config.download_folder, item)
        if os.path.isfile(item_path):
            os.remove(item_path)

# 開始下載
print(" ")
print("開始下載所有文章...")
for index, journal in enumerate(journals):

    print(f"---------- {index + 1}/{len(journals)} -----------")
    print(f"DOI: {journal.doi}")
    print(f"日期: {journal.created_date}")
    print(f"標題: {journal.title}")
    print(f"作者: {journal.author}")
    print("正在下載...")

    # 檢查 DOI、標題和作者是否為 'NULL'
    if(journal.title == 'NULL' or journal.author == 'NULL'):
        journal.result = 'No Title or Author'
        print(f"下載結果: {journal.download_result}")
    else:
        tdm = TDMClient(config.apikey)
        tdm.download_dir =  config.download_folder
        journal.download_result = str(tdm.download_pdf(journal.doi))
        print(f"下載結果: {journal.download_result}")
        
        if(journal.download_result != 'Success'): # 若下載失敗 暫停後再次下載
            sleep(300)  # 暫停 300 秒
            journal.download_result = tdm.download_pdf(journal.doi)
            print(f"再次下載結果: {journal.download_result}")
        else :
            sleep(1)  # 暫停 1 秒

        # 修改檔名
        old_name = os.path.join(tdm.download_dir , journal.download_file_org_name)
        new_name = os.path.join(tdm.download_dir, journal.download_file_new_name)
        if(os.path.isfile(old_name)):
            os.rename(old_name, new_name)
            print(f"檔名重新名完畢：{journal.download_file_org_name} -> {journal.download_file_new_name}")

print("--------------------------")

# 將結果儲存成CSV
csvjournals = []
for index, journal in enumerate(journals):
    if(index == 0):
        csvjournals.append(['DOI', '日期', '標題', '作者', '下載狀態', '下載檔名', '重新命名檔名'])
    csvjournals.append([journal.doi, journal.created_date, journal.title, journal.author, journal.download_result, journal.download_file_org_name, journal.download_file_new_name])
csv_file = os.path.join(config.download_folder, 'journals.csv')
with open(csv_file, 'w', newline='', encoding='utf-8') as f:
    writer = csv.writer(f)
    writer.writerows(csvjournals)
print(" ")
print("儲存文章資料及下載狀態為CSV完畢...")

# 完成訊息
print(" ")
print("下載程序結束...")

```
