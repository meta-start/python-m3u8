# python-m3u8

### python download m3u8 and convert to mp4. 添加代理, 解密视频, 下载m3u8

#### 1 获取ts_url列表

##### 1.1 通过返回的元组resp_tuple判断并解析m3u8地址

```python
 def get_urls(self, url):
        """
        获取response中的urls, keys
        :param url: requests请求的url地址 (原m3u8_url和解析后m3u8_url)
        :return: 返回list, keys
        """
        response = self.session.get(url, headers=self.headers)
        lines = response.text.strip().split('\n')
        urls, keys = [], []
        for line in lines:
            if line.endswith('.ts'):
                urls.append(line)
            elif line.endswith('.m3u8'):
                urls.append(line)
            elif line.startswith('#EXT-X-KEY'):
                keys.append(line)
        return urls, keys
```

##### 1.2 解析地址

```python
@staticmethod
    def parse_url(m3u8_url, tail_url):
        """
        解析单行正确的url
        :param m3u8_url: 请求的地址,用来获取基准url
        :param tail_url: response的单行url
        :return: 正确的url, flag表示解析类型,基准url,url字符串去重
        """
        head_url = m3u8_url.rpartition('/')[0]
        end_with_str = ''
        for tail_str in tail_url.split('/'):
            if tail_str != '' and head_url.endswith(tail_str):
                end_with_str = tail_str
                break
        if end_with_str:
            url = head_url + tail_url.partition(end_with_str)[2]
            flag = 2
        else:
            if tail_url.startswith('/'):
                url = head_url + tail_url
                flag = 1
            else:
                url = head_url + '/' + tail_url
                flag = 0
        return url, flag, head_url, end_with_str
```



##### 1.3 判断是否加密

```python
    def is_crack(self, m3u8_url, keys):
        """
        判断是否为加密视频
        :param m3u8_url: m3u8_url
        :param keys: #EXT-X-KEY
        :return:
        """
        if keys:
            print('\rstart crack...', end='')
            self.is_crypt = True
            ext_x_key = keys[0]
            uri = re.search('URI=\"(.*?)\"', ext_x_key).group(1)
            key_url = M3U8.parse_url(m3u8_url, uri)[0]
            key = self.session.get(key_url, headers=self.headers).content
            if re.search('IV=(.*)', ext_x_key):
                iv = re.search('IV=(.*)', ext_x_key).group(1).replace('0x', "")[:16].encode()
            else:
                iv = b'0000000000000000'
            self.to_crack = AES.new(key, AES.MODE_CBC, iv)
            print('\r' + 'start crack'.ljust(36, '.') + 'done\n', end='')
```



##### 1.4 解析出ts_url

```python
    def get_ts_urls(self):
        """
        获取ts_url列表
        :return: ts_list
        """
        print('\rparse m3u8 url...', end='')
        resp_tuple = self.get_urls(self.m3u8_url)
        urls = resp_tuple[0]
        if len(urls) == 1:
            m3u8_url = M3U8.parse_url(self.m3u8_url, urls[0])[0]
            m3u8_resp_tuple = self.get_urls(m3u8_url)
            ts_list = m3u8_resp_tuple[0]
        else:
            m3u8_url = self.m3u8_url
            m3u8_resp_tuple = resp_tuple
            ts_list = urls
        print('\r' + 'parse m3u8 url'.ljust(36, '.') + 'done\n', end='')
        keys = m3u8_resp_tuple[1]
        self.is_crack(m3u8_url, keys)
        print('\rfetch ts url...', end='')
        ts0 = ts_list[0]
        ts_urls = []
        if ts0.startswith('http'):
            ts_urls = ts_list
        else:
            ts0_tuple = M3U8.parse_url(m3u8_url, ts0)
            flag = ts0_tuple[1]
            head_url = ts0_tuple[2]
            end_with_str = ts0_tuple[3]
            if flag == 0:
                for ts_url in ts_list:
                    ts_u = head_url + '/' + ts_url
                    ts_urls.append(ts_u)
            elif flag == 1:
                for ts_url in ts_list:
                    ts_u = head_url + ts_url
                    ts_urls.append(ts_u)
            else:
                for ts_url in ts_list:
                    ts_u = head_url + ts_url.partition(end_with_str)[2]
                    ts_urls.append(ts_u)
        print('\r' + 'fetch ts url'.ljust(36, '.') + 'done\n', end='')
        return ts_urls
```



#### 2 开启线程池下载ts

#### 3.合并解密后的ts文件
