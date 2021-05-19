# coding:utf-8
import os
import requests
import threading
from bs4 import BeautifulSoup
import time
import math

request_headers = {
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'Accept-Encoding': 'gzip,deflate',
    'Accept-Language': 'zh-CN,zh;q=0.9',
    'Connection': 'keep-alive',
    'Cookie': '_abcde_qweasd=0; bdshare_firstime=1620973163314; BAIDU_SSP_lcr=https://www.baidu.com/link?url=Gd2io5yF4DvhB8ZGnNyEmfWVP-IWhQLfvqwJ8Ju8E6S&wd=&eqid=f85cb3fd0002539f0000000660a36228; _abcde_qweasd=0; Hm_lvt_169609146ffe5972484b0957bd1b46d6=1620973163,1620973671,1621320236; Hm_lpvt_169609146ffe5972484b0957bd1b46d6=1621320307; cscpvcouplet9193_fidx=1; cscpvrich9192_fidx=4',
    'Host': 'www.xbiquge.la',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.212 Safari/537.36'
}

request_url_base = 'http://www.xbiquge.la/'

# 将文章信息写入txt文件
def write_txt(content, title):
    with open(title + '.txt', 'a', encoding='utf-8') as file:
        file.write(content + "\n\n\n")

# 文件合并
def file_merge(title):
    for i in range(4):
        file_name = '{}_{}.txt'.format(title, i)
        with open(file_name, 'r', encoding='utf-8') as file:
            write_txt(file.read(), title)
        # 合并后删除临时文件
        os.remove(file_name)

# 将列表分成n块
def reshape(listTemp, n):
    num = math.ceil(len(listTemp) / 4)
    for i in range(0, len(listTemp), num):
        yield listTemp[i:i + num]

# 获取小说章节内容
def get_chapter(obj, novel, index):
    try:
        for key,i in enumerate(obj):
            title = i.get_text()
            chapter = requests.get(request_url_base.strip('/') + str(i['href']), params=request_headers)
            chapter.encoding = 'utf-8'
            soup = BeautifulSoup(chapter.text, 'html.parser')

            title = soup.select('#wrapper .content_read .box_con .bookname h1')[0].get_text()
            content = soup.select('#wrapper .content_read .box_con #content')[0]
            content.p.decompose()
            content = content.get_text().replace('<br>', '\r\n')

            # 章节内容写进临时文件
            content = "{}\r\n{}".format(title, content)
            write_txt(content, novel + '_' + str(index))
            print("{} {} ...... 下载完成".format(novel, title))

    except Exception as e:
        print("{} {} ...... 下载失败".format(novel, title))

# 获取小说信息
# book:小说在新笔趣阁的编号
# title:小说名称
# category:小说类型
# desc：小说描述
def get_txt(book):
    info = {}
    try:
        time1 = int(time.time())
        # 小说目录页信息
        request_url = request_url_base.strip('/') + '/2/' + str(book) + '/'
        res = requests.get(request_url, params=request_headers)
        res.encoding = 'utf-8' # 网页编码
        soups = BeautifulSoup(res.text, 'html.parser')

        # 小说信息
        info['title'] = soups.select('meta[property="og:title"]')[0]['content']
        info['author'] = soups.select('meta[property="og:novel:author"]')[0]['content']
        info['category'] = soups.select('meta[property="og:novel:category"]')[0]['content']
        info['desc'] = soups.select('meta[property="og:description"]')[0]['content']
        print("小说：《{}》 开始下载".format(info['title']))
        print("作者：{}".format(info['author']))

        # 将小说信息写入本地文件
        content = '''小说：{title}
作者：{author}
小说类型：{category}
/*********** 简介 ***********/
{desc}
/***************************/
'''.format(**info)
        write_txt(content, info['title'])

        # 小说章节
        catalogue = soups.select('#wrapper .box_con #list dl dd a')
        catalogue = reshape(catalogue, 4)
        thread_list = []
        for index, infor in enumerate(catalogue):
            thread_list.append(threading.Thread(target=get_chapter, args=(infor, info['title'], index)))

        for th in thread_list:
            th.start()

        for th in thread_list:
            th.join()

        # 文件合并
        file_merge(info['title'])
        print("小说：《{}》 下载结束，耗时：{}秒".format(info['title'], (int(time.time()) - time1)))

    except Exception as e:
        print("小说读取失败！")
        error_info = '[{}] 编号：{} 下载失败.'.format(time.strftime('%Y-%m-%d %X', time.localtime()), book)
        write_txt(error_info, 'downError')

get_txt('2210')
