pytchat
=======

pytchat is a python library for fetching youtube live chat.

## Description
pytchat is a python library for fetching youtube live chat
without using youtube api, Selenium or BeautifulSoup.

pytchatはAPIを使わずにYouTubeチャットを取得するためのpythonライブラリです。

Other features:
+ Customizable [chat data processors](https://github.com/taizan-hokuto/pytchat/wiki/ChatProcessor) including youtube api compatible one.
+ Available on asyncio context. 
+ Quick fetching of initial chat data by generating continuation params
instead of web scraping.

For more detailed information, see [wiki](https://github.com/taizan-hokuto/pytchat/wiki). <br>
より詳細な解説は[wiki](https://github.com/taizan-hokuto/pytchat/wiki/Home_jp)を参照してください。

## Install
```python
pip install pytchat
```
## Examples

### CLI

One-liner command.
Save chat data to html.

```bash
$ pytchat -v ZJ6Q4U_Vg6s -o "c:/temp/"

# options:
#  -v : video_id
#  -o : output directory (default path: './')
# saved filename is [video_id].html
```


### on-demand mode
```python
from pytchat import LiveChat
livechat = LiveChat(video_id = "Zvp1pJpie4I")

while livechat.is_alive():
  try:
    chatdata = livechat.get()
    for c in chatdata.items:
        print(f"{c.datetime} [{c.author.name}]- {c.message}")
        chatdata.tick()
  except KeyboardInterrupt:
    livechat.terminate()
    break
```

### callback mode
```python
from pytchat import LiveChat
import time

def main():
  livechat = LiveChat(video_id = "Zvp1pJpie4I", callback = disp)
  while livechat.is_alive():
    #other background operation.
    time.sleep(1)
  livechat.terminate()

#callback function (automatically called)
def disp(chatdata):
    for c in chatdata.items:
        print(f"{c.datetime} [{c.author.name}]- {c.message}")
        chatdata.tick()

if __name__ == '__main__':
  main()

```

### asyncio context:
```python
from pytchat import LiveChatAsync
from concurrent.futures import CancelledError
import asyncio

async def main():
  livechat = LiveChatAsync("Zvp1pJpie4I", callback = func)
  while livechat.is_alive():
    #other background operation.
    await asyncio.sleep(3)

#callback function is automatically called.
async def func(chatdata):
  for c in chatdata.items:
    print(f"{c.datetime} [{c.author.name}]-{c.message} {c.amountString}")
    await chatdata.tick_async()

if __name__ == '__main__':
  try:
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
  except CancelledError:
    pass
```


### youtube api compatible processor:
```python
from pytchat import LiveChat, CompatibleProcessor
import time

chat = LiveChat("Zvp1pJpie4I", 
  processor = CompatibleProcessor() )

while chat.is_alive():
  try:
    data = chat.get()
    polling = data['pollingIntervalMillis']/1000
    for c in data['items']:
      if c.get('snippet'):
        print(f"[{c['authorDetails']['displayName']}]"
              f"-{c['snippet']['displayMessage']}")
        time.sleep(polling/len(data['items']))
  except KeyboardInterrupt:
    chat.terminate()
```
### replay: 
If specified video is not live,
automatically try to fetch archived chat data.

```python
from pytchat import LiveChat

def main():
  #seektime (seconds): start position of chat.
  chat = LiveChat("ojes5ULOqhc", seektime = 60*30)
  print('Replay from 30:00')
  try:
    while chat.is_alive():
      data = chat.get()
      for c in data.items:
        print(f"{c.elapsedTime} [{c.author.name}]-{c.message} {c.amountString}")
        data.tick()
  except KeyboardInterrupt:
    chat.terminate()

if __name__ == '__main__':
  main()
```

## Structure of Default Processor
Each item can be got with `items` function.
<table>
  <tr>
    <th>name</th>
    <th>type</th>
    <th>remarks</th>
  </tr>
  <tr>
    <td>type</td>
    <td>str</td>
    <td>"superChat","textMessage","superSticker","newSponsor"</td>
  </tr>
  <tr>
    <td>id</td>
    <td>str</td>
    <td></td>
  </tr>
  <tr>
    <td>message</td>
    <td>str</td>
    <td>emojis are represented by ":(shortcut text):"</td>
  </tr>
  <tr>
    <td>messageEx</td>
    <td>str</td>
    <td>list of message texts and emoji URLs.</td>
  </tr>
  <tr>
    <td>timestamp</td>
    <td>int</td>
    <td>unixtime milliseconds</td>
  </tr>
  <tr>
    <td>datetime</td>
    <td>str</td>
    <td>e.g. "2019-10-10 12:34:56"</td>
  </tr>
    <td>elapsedTime</td>
    <td>str</td>
    <td>elapsed time. (e.g. "1:02:27") *Replay Only.</td>
  </tr>
  <tr>
    <td>amountValue</td>
    <td>float</td>
    <td>e.g. 1,234.0</td>
  </tr>
  <tr>
    <td>amountString</td>
    <td>str</td>
    <td>e.g. "$ 1,234"</td>
  </tr>
  <tr>
    <td>currency</td>
    <td>str</td>
    <td><a href="https://en.wikipedia.org/wiki/ISO_4217">ISO 4217 currency codes</a> (e.g. "USD")</td>
  </tr>
  <tr>
    <td>bgColor</td>
    <td>int</td>
    <td>RGB Int</td>
  </tr>
  <tr>
    <td>author</td>
    <td>object</td>
    <td>see below</td>
  </tr>
</table>

Structure of author object.
<table>
  <tr>
    <th>name</th>
    <th>type</th>
    <th>remarks</th>
  </tr>
  <tr>
    <td>name</td>
    <td>str</td>
    <td></td>
  </tr>
  <tr>
    <td>channelId</td>
    <td>str</td>
    <td>*chatter's channel ID.</td>
  </tr>
  <tr>
    <td>channelUrl</td>
    <td>str</td>
    <td></td>
  </tr>
  <tr>
    <td>imageUrl</td>
    <td>str</td>
    <td></td>
  </tr>
  <tr>
    <td>badgeUrl</td>
    <td>str</td>
    <td></td>
  </tr>
  <tr>
    <td>isVerified</td>
    <td>bool</td>
    <td></td>
  </tr>
  <tr>
    <td>isChatOwner</td>
    <td>bool</td>
    <td></td>
  </tr>
  <tr>
    <td>isChatSponsor</td>
    <td>bool</td>
    <td></td>
  </tr>
  <tr>
    <td>isChatModerator</td>
    <td>bool</td>
    <td></td>
  </tr>
</table>

## Licence

[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)


## Contributes
Great thanks:

Most of source code of CLI refer to:

[PetterKraabol / Twitch-Chat-Downloader](https://github.com/PetterKraabol/Twitch-Chat-Downloader)


## Author

[taizan-hokuto](https://github.com/taizan-hokuto)

[twitter:@taizan205](https://twitter.com/taizan205)
