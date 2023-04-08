# python-m3u8

### python download m3u8 and convert to mp4.

#### 1 获取ts_url列表

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

##### 1.1 通过返回的元组resp_tuple判断并解析m3u8地址

##### 1.2 通过m3u8地址获取ts以及key信息

##### 1.3 解析出ts_url并判断是否加密

#### 2 开启线程池下载ts

#### 3.合并解密后的ts文件
