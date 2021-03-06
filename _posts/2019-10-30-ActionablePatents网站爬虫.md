---
layout:     post           # 使用的布局（不需要改）
title:      ActionablePatents网站爬虫           # 标题 
subtitle:   ActionablePatents网站爬虫 #副标题
date:       2019-10-30             # 时间
author:     甜果果                    # 作者
header-img: https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@1.0/assets/img/post-bg-ios9-web.jpg    #背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 技术
    - 爬虫
    - python

---

# ActionablePatents网站爬虫

使用actionalpatents网站的高级检索功能，选中“chinese patents”和“Chinese utilities”，可以搜出中英文的专利文本。

![image-20200725152821209](https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@master/assets/picgoimg/20200725152821.png)

以“新能源”为关键词的搜索结果

![image-20200725153823356](https://cdn.jsdelivr.net/gh/tian-guo-guo/cdn@master/assets/picgoimg/20200725153823.png)

使用Selenium来爬取标题、摘要和权力要求书

```python
from selenium import webdriver
import time
import os

web = 'http://www.actionablepatents.com/Search/Workboard/AdvancedHome#Form'

class crawler(object):
    def __init__(self, key_word="", page_start=1, page_end=5):
        self.Key_word = key_word  
        self.page_current = 1
        self.start = page_start
        self.page_start = self.start
        self.page_end = page_end
        self.times = 0

    def save_txt(self, current, number, path, patent):
        filename = patent['Patent_Num'] + '_' + str(current) + '_' + str(number) + ".txt"
        path_all = os.path.join(path, filename)
        f = open(path_all, 'a', encoding='utf-8')
        print('写入txt' + str(current) + '_' + str(number))
        for key in patent.keys():
            content = patent[key].strip()
            f.write(key + '\t' + content + '\r\n')
        f.close()

    def login(self):
        # driver = webdriver.Chrome(executable_path='E:\chromedriver\chromedriver.exe')   # windows
        
        driver = webdriver.Chrome(executable_path='/usr/local/bin/chromedriver')  # linux

        driver.get(web)
        # 窗口最大化 要不显示不全不行
        
        driver.maximize_window()
        time.sleep(5)

        # driver.find_elements_by_id("SearchForm_Category")[1].click()

        driver.find_elements_by_id("SearchForm_Category")[0].click()
        driver.find_elements_by_id("SearchForm_Category")[1].click()
        driver.find_elements_by_id("SearchForm_Category")[2].click()
        driver.find_elements_by_id("SearchForm_Category")[3].click()
        # 对申请专利爬取
        
        # driver.find_elements_by_id("SearchForm_GrantedChkBox").click()

        # # 对批准专利爬取
        
        # driver.find_element_by_id("SearchForm_PublicationChkBox").click()

        driver.find_element_by_id("SearchForm_SimpleSearch_SimpleQuery").send_keys(self.Key_word)
        driver.find_element_by_id("slideToggleSmallButton").click()

        # driver.find_elements_by_class_name('menuNormal')[1].click()
        
        # driver.find_element_by_id('UserId').send_keys('525322746@qq.com')
        
        # driver.find_element_by_id('Password').send_keys('icdd123')
        #
        
        # driver.find_element_by_class_name('btn_popup_login').click()
        
        # time.sleep(5)
        
        # driver.find_element_by_xpath('/ html / body / div[14] / div[3] / div / button[1]').click()
        
        time.sleep(10)

        self.times = 0
        try:
            for page_current in range(self.page_start, self.page_end):
                time.sleep(2)
                print("page_current:" + str(page_current))
                driver.find_element_by_id("CurrentInfo_listPageInput").clear()
                driver.find_element_by_id("CurrentInfo_listPageInput").send_keys(page_current)
                time.sleep(5)
                driver.find_element_by_id("moveListPage").click()
                self.times += 1
                driver.implicitly_wait(20)
                time.sleep(10)

                hrefs = driver.find_elements_by_class_name('patentno')

                i = 0
                for href in hrefs:
                    href.click()
                    driver.implicitly_wait(30)
                    time.sleep(5)
                    patent_data = driver.find_elements_by_class_name("custom-highlight-area")[12:]

                    patent = {}
                    patent['Patent_Num'] = patent_data[0].text
                    patent['Patent_english'] = patent_data[1].text
                    patent['Patent_chinese'] = patent_data[2].text
                    patent['Patent_status'] = patent_data[3].text
                    if 'Alive' in patent_data[3].text:
                        patent['Patent_termination'] = patent_data[5].text.split(':')[-1]
                        patent['Patent_grade'] = patent_data[6].text
                        patent['Patent_abstract_english'] = patent_data[7].text
                        patent['Patent_abstract_chinese'] = patent_data[8].text
                    else:
                        patent['Patent_termination'] = '空'
                        patent['Patent_grade'] = patent_data[5].text
                        patent['Patent_abstract_english'] = patent_data[6].text
                        patent['Patent_abstract_chinese'] = patent_data[7].text

                    patent_claim = driver.find_elements_by_class_name("USCLAIMTEXT")
                    patent['Patent_claim_english'] = patent_claim[0].text
                    patent['Patent_claim_chinese'] = patent_claim[1].text

                    aa = driver.find_elements_by_xpath('//*[@id="FullTextDiv"]/div[3]/table[2]/tbody/tr/th')
                    bb = driver.find_elements_by_xpath('//*[@id="FullTextDiv"]/div[3]/table[2]/tbody/tr/td')
                    for a, b in zip(aa, bb):
                        b_list = b.text.strip().split('\n')
                        b_str = ''
                        for j in b_list:
                            b_str += j + ';'
                        patent[a.text] = b_str

                    patent['Patent_claim_num'] = driver.find_element_by_id("btClaims").text.split('/')[-1][:-1]

                    patent['Patent_Inpadoc_family'] = driver.find_element_by_id('divCFamily').text.split('(')[1].strip().split()[0]
                    patent['Patent_Simple_family'] = driver.find_element_by_id('divCSimpleFamily').text.split('(')[1].strip().split()[0]
                    patent['Patent_Citation'] = driver.find_element_by_id('divCDREF').text.split('(')[1].strip().split()[0]
                    patent['Patent_Cited'] = driver.find_element_by_id('divCDREFBY').text.split('(')[1].strip().split()[0]
                    patent['Patent_Foreign_Ref'] = driver.find_element_by_id('divCFREF').text.split('(')[1].strip().split()[0]
                    patent['Patent_Ref_Cited'] = driver.find_element_by_id('divCOREF').text.split('(')[1].strip().split()[0]

                    # claim_click = driver.find_element_by_id('btClaims')
                    
                    # claim_click.click()
                    
                    time.sleep(3)

                    # patent['Patent_claim'] = driver.find_element_by_class_name("normalClaims").text

                    i += 1
                    self.page_current = page_current+1
                    self.save_txt(path="专利新能源/patent_新能源/patent",
                                  current=page_current, number=i, patent=patent)
                    time.sleep(2)
        except Exception as e:
            print(e)
            driver.close()
            driver.quit()
            self.page_start = self.page_current
            time.sleep(5)
            self.login()

if __name__ == '__main__':
    crawler = crawler(page_start=2, page_end=3)
    crawler.login()
```



