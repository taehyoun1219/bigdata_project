import requests
import bs4
import pandas as pd
from selenium.webdriver import Chrome
import time
from tqdm.notebook import tqdm
import warnings
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
warnings.simplefilter('ignore')
import dash
from dash.dependencies import Input, Output
from dash import dcc
from dash import html

query = "애플"

titles = []
prices = []
review_counts = []
buy_counts = []
published_dates = []
favorites = []

chrome_options = Options()
chrome_options.add_argument('--headless')
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')
chrome_options.add_argument('--disable-gpu')
chrome_options.add_argument(f"--window-size=1920,1080")

# Chrome 드라이버는 이미 설치되어 있으므로 별도로 경로를 지정할 필요가 없습니다.

driver = webdriver.Chrome(options=chrome_options)

for page_no in tqdm(range(1,6)):
  page_url = f"https://search.shopping.naver.com/search/all?origQuery={query}&pagingIndex={page_no}&pagingSize=40&productSet=total&query={query}&sort=rel&timestamp=&viewType=list"
  driver.get(page_url)
  time.sleep(1)

  for scroll_down in range(7):
    driver.execute_script('window.scrollTo(0,document.body.scrollHeight);')
    time.sleep(0.5)

  list_basis = driver.find_element(By.CLASS_NAME,"basicList_list_basis__uNBZx")
  item_list = list_basis.find_elements(By.CLASS_NAME,'product_item__MDtDF')

  for i in tqdm(range(len(item_list))):
    item = item_list[i]
    title = item.find_element(By.CLASS_NAME,"product_title__Mmw2K")
    titles.append(title.text)

    price = item.find_element(By.CLASS_NAME,"price_num__S2p_v").text[:-1].replace(',','')
    prices.append(price)

    footer = item.find_element(By.CLASS_NAME,"product_etc_box__ElfVA")
    reviews = footer.find_elements(By.CLASS_NAME,'product_etc__LGVaW')
    footer_text = footer.text

    try:
      if '구매건수' in footer_text:
        review1 = reviews[0].text
        #print(review1)
        if '구매건수' in review1:
          review_counts.append(0)
          buy_counts.append(0)
          favorites.append(0)
          published_dates.append(0)
          continue
        review_counts.append(int(review1[2:].replace(',','')))
        buy_counts1 = reviews[1].text
        buy_counts.append(int(buy_counts1[4:].replace(',','')))
        favorites1 = reviews[2].text
        favorites.append(int(favorites1[4:].replace('.','')))
        date = footer.find_elements(By.TAG_NAME,"span")[0].text[4:]
        date1 = footer.find_elements(By.TAG_NAME,"span")[0].text
        published_dates.append(date)
      else:
        year = reviews[2].text
        #print(year)
        if year == '신고하기' or year == '찜하기' or year == '수정요청':
          review_counts.append(0)
          published_dates.append(0)
          favorites.append(0)
          continue
        favorites.append(int(year[3:]))
        review_counts.append(0)
        date = footer.find_elements(By.CLASS_NAME,"product_etc__LGVaW")[1].text[4:]
        published_dates.append(date)


    except IndexError:
      review_counts.append(0)
      if reviews:
        favorites.append(int(reviews[0].text.replace(',', '')))
      else:
        favorites.append(0)
      #date = footer.find_elements(By.TAG_NAME,"span")[0].text[4:]
      published_dates.append(0)



result = pd.DataFrame({
    "제품명" : titles,
    "가격" : prices,
    "리뷰수" : review_counts,
    "구매건수" : published_dates,
    "등록일" : favorites
})

df = result[result['리뷰수'] != 0]


# dashboard app
app = dash.Dash('Naver Shopping Trend',
                external_stylesheets=['https://codepen.io/chriddyp/pen/bWLwgP.css'])

# app layout-> html 수정.
app.layout = html.Div([
    dcc.Dropdown(
        id='my-dropdown',
        options=[
            {'label':'Apple', 'value':'/content/naver_shopping(애플).xlsx'}
            #{'label':'Samsung', 'value':'./naver_shopping(삼성전자).xlsx'},
            #{'label':'Xiaomi', 'value':'./naver_shopping(샤오미).xlsx'}
        ],
        value= '/content/naver_shopping(애플).xlsx'
    ),
    dcc.Graph(id='my-graph')
], style={'width': '600'})


@app.callback(Output('my-graph', 'figure'), [Input('my-dropdown', 'value')])
# dash가 실제로 실행하는 그래프를 update_graph 함수로 정의합니다.
def update_graph(selected_dropdown_value):
    # 내가 선택한 label에 해당하는 파일 이름

    return {
        'data': [
           {'x':df.index, 'y':df["리뷰수"]}

        ],
        'layout': {'margin': {'l': 40, 'r': 0, 't': 20, 'b': 30}}
    }

# dash app이 실행됩니다.
app.run_server(debug=True, use_reloader=False)
#app.run_server(host='192.168.0.3', port=3003)

