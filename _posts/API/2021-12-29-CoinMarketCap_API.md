---
layout: single
title: "[API] Coinmarketcap API"
categories:
  - API
tags:
  - API
  - python
  - coinmarketcap
toc: true
toc_sticky: true
toc_label: "CoinMarketCap API"
toc_icon: "file-code"
---

💡 러그풀 탐지 AI 모델을 개발하는 프로젝트에서 여러 코인들의 정보가 필요한적이 있었습니다.<br>
CoinmarketCap은 코인에 대한 정보들을 리스팅한 가장 큰 사이트로, 해당 사이트의 API 사용 방법을 정리한 글입니다.
{: .notice--warning}

# CoinMarketCap??

![image](https://user-images.githubusercontent.com/33647663/148165944-423267fc-28f6-455e-8586-a3ab055a46fa.jpg){: .align-center}

API를 사용하기 위해선 CoinMarketCap에 회원가입을 해서 API키를 발급받으셔야 하며 API키는  [링크](https://coinmarketcap.com/api/pricing/) 에서 받을 수 있습니다. 또한 API 사용법에 대한 공식문서는 [이곳]([Account](https://coinmarketcap.com/api/documentation/v1/#section/Quick-Start-Guide))을 참조하시면 됩니다.

모든 API 기능에 대해서 전부 설명하지는 않고,  무료 API키로 사용가능한 기능 몇가지에 대해서만 관련 정보와 실행 결과에 대해서만 정리하겠습니다.

## Cryptocurrency/listings/latest

- 현재 거래되고 있는(코인마켓 캡에 리스팅 되어있는)모든 토큰들에 대한 전반적인 정보들을 반환합니다.
  
  ```
  [ 반환 값 예]
  - id : 825
  - name : Tether
  - symbol : USDT
  - slug : tether
  - num_market_pairs : 17649
  - date_added:"2015-"
  - tags : payments/stablecoin/bsc/solana-ecosystem
  - max_supply/ circulating_supply/ total_supply
  - platform : {"name" : ether, "token_add" : 0x11111..}
  - quote { "USD" : {가격/24시간 볼륨/1시간 변화량/ 24시간 변화량/7일 변화량
    30일 변화량/ 60일 변화량/ 90일 변화량 }}
  ```
  
  ```python
  #This example uses Python 2.7 and the python-request library.
  #제공해주는 예제 코드
  #아래의 url로 쿼리를 날려서 정보를 얻어온다.
  #코인마켓캡 기준으로 1번 코인부터 249번까지의 코인 정보를 받아옴
  #https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest?start=1&limit:250&convert=USD
  
  from requests import Request, Session
  from requests.exceptions import ConnectionError, Timeout, TooManyRedirects
  import pandas as pd
  import json
  
  url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest'
  parameters = {
    'start':'1',
    'limit':'249',
    'convert':'USD',
    'aux':'num_market_pairs,cmc_rank'
  }
  
  #API_KEY는 자신의 API키로 바꿔줘야 한다.
  headers = {
    'Accepts': 'application/json',
    'X-CMC_PRO_API_KEY': '자신의 API 키',
  }
  
  session = Session()
  session.headers.update(headers)
  
  try:
    response = session.get(url, params=parameters)
    datas = json.loads(response.text)
    print(datas)
  except (ConnectionError, Timeout, TooManyRedirects) as e:
    print(e)
  ```

## cryptocurrency/map

* 코인에 해당하는 ID값을 반환합니다. [ ID는 코인마켓캡 내에서 코인들을 식별하는데 이용]
  
  

  ```python
  #This example uses Python 2.7 and the python-request library.
  #제공해주는 예제 코드
  #아래의 url로 쿼리를 날려서 정보를 얻어온다.
  #코인마켓캡 기준으로 1번 코인부터 249번까지의 코인 정보를 받아옴
  #https://pro-api.coinmarketcap.com/v1/cryptocurrency/listings/latest?start=1&limit:250&convert=USD

  from requests import Request, Session
  from requests.exceptions import ConnectionError, Timeout, TooManyRedirects
  import json

  url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/map'
  parameters = {
    #'listing_status':'inactive',
    #'limit':'249',
    #'convert':'USD'
  }

  #API_KEY는 자신의 API키로 바꿔줘야 한다.
  headers = {
    'Accepts': 'application/json',
    'X-CMC_PRO_API_KEY': '자신의 API키',
  }

  session = Session()
  session.headers.update(headers)

  try:
    response = session.get(url, params = parameters)
    data = json.loads(response.text)
    print(data)
  except (ConnectionError, Timeout, TooManyRedirects) as e:
    print(e)
  ```

## cryptocurrency/cryptocurrency/info

  1. id를 주면 해당하는 코인의 정보를 받아올 수 있음
  2. input으로 slug를 받아서 정보 출력 구현 완료

  
  ```python
  from requests import Request, Session
  from requests.exceptions import ConnectionError, Timeout, TooManyRedirects
  import json

  def info_id(id):
    url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/info'
    parameters = {
      #'slug':slug,
      #'limit':'249',
      #'convert':'USD'
      #'aux' : 'urls,description,logo'
      'id':id
    }

  def info_slug(slug):
    url = 'https://pro-api.coinmarketcap.com/v1/cryptocurrency/info'
    parameters = {
      'slug':slug,
      #'limit':'249',
      #'convert':'USD'
      #'aux' : 'urls,description,logo'
    }


  #API_KEY는 자신의 API키로 바꿔줘야 한다.
    headers = {
      'Accepts': 'application/json',
      'X-CMC_PRO_API_KEY': '자신의 API키',
    }

    session = Session()
    session.headers.update(headers)

    try:
      response = session.get(url, params = parameters)
      data = json.loads(response.text)
  #    print(data)
      return data
    except (ConnectionError, Timeout, TooManyRedirects) as e:
      print(e)

    slug_name = input()
    ret = info_slug(slug_name)

    id = list(ret['data'].keys())[0]
    technical_doc = ret['data'][id]['urls']['technical_doc']
    source_code = ret['data'][id]['urls']['source_code']
    website = ret['data'][id]['urls']['website']
    description = ret['data'][id]['description']
    logo = ret['data'][id]['logo']
    tags = ret['data'][id]['tags']

    print("백서 :" +str(technical_doc))
    print("소스 :" +str(source_code))
    print("웹사이트 :" +str(website))
    print("설명 :" +str(description))
    print("logo :" +str(logo))
    print("tags :" +str(tags))
  ```

  [ 코드 실행 결과]
  ```
    bitcoin
    {'status': {'timestamp': '2021-09-10T06:26:25.332Z', 'error_code': 0, 'error_message': None, 'elapsed': 16, 'credit_count': 1, 'notice': None}, 'data': {'1': {'id': 1, 'name': 'Bitcoin', 'symbol': 'BTC', 'category': 'coin', 'description': 'Bitcoin (BTC) is a cryptocurrency . Users are able to generate BTC through the process of mining. Bitcoin has a current supply of 18,811,693. The last known price of Bitcoin is 46,614.98711064 USD and is up 0.94 over the last 24 hours. It is currently trading on 8840 active market(s) with $35,731,483,458.68 traded over the last 24 hours. More information can be found at https://bitcoin.org/.', 'slug': 'bitcoin', 'logo': 'https://s2.coinmarketcap.com/static/img/coins/64x64/1.png', 'subreddit': 'bitcoin', 'notice': '', 'tags': ['mineable', 'pow', 'sha-256', 'store-of-value', 'state-channels', 'coinbase-ventures-portfolio', 'three-arrows-capital-portfolio', 'polychain-capital-portfolio', 'binance-labs-portfolio', 'arrington-xrp-capital', 'blockchain-capital-portfolio', 'boostvc-portfolio', 'cms-holdings-portfolio', 'dcg-portfolio', 'dragonfly-capital-portfolio', 'electric-capital-portfolio', 'fabric-ventures-portfolio', 'framework-ventures', 'galaxy-digital-portfolio', 'huobi-capital', 'alameda-research-portfolio', 'a16z-portfolio', '1confirmation-portfolio', 'winklevoss-capital', 'usv-portfolio', 'placeholder-ventures-portfolio', 'pantera-capital-portfolio', 'multicoin-capital-portfolio', 'paradigm-xzy-screener'], 'tag-names': ['Mineable', 'PoW', 'SHA-256', 'Store of Value', 'State channels', 'Coinbase Ventures Portfolio', 'Three Arrows Capital Portfolio', 'Polychain Capital Portfolio', 'Binance Labs Portfolio', 'Arrington XRP capital', 'Blockchain Capital Portfolio', 'BoostVC Portfolio', 'CMS Holdings Portfolio', 'DCG Portfolio', 'DragonFly Capital Portfolio', 'Electric Capital Portfolio', 'Fabric Ventures Portfolio', 'Framework Ventures', 'Galaxy Digital Portfolio', 'Huobi Capital', 'Alameda Research Portfolio', 'A16Z Portfolio', '1Confirmation Portfolio', 'Winklevoss Capital', 'USV Portfolio', 'Placeholder Ventures Portfolio', 'Pantera Capital Portfolio', 'Multicoin Capital Portfolio', 'Paradigm XZY Screener'], 'tag-groups': ['OTHER', 'CONSENSUS_ALGORITHM', 'CONSENSUS_ALGORITHM', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY', 'PROPERTY'], 'urls': {'website': ['https://bitcoin.org/'], 'twitter': [], 'message_board': ['https://bitcointalk.org'], 'chat': [], 'explorer': ['https://blockchain.coinmarketcap.com/chain/bitcoin', 'https://blockchain.info/', 'https://live.blockcypher.com/btc/', 'https://blockchair.com/bitcoin', 'https://explorer.viabtc.com/btc'], 'reddit': ['https://reddit.com/r/bitcoin'], 'technical_doc': ['https://bitcoin.org/bitcoin.pdf'], 'source_code': ['https://github.com/bitcoin/'], 'announcement': []}, 'platform': None, 'date_added': '2013-04-28T00:00:00.000Z', 'twitter_username': '', 'is_hidden': 0}}}
    백서 :['https://bitcoin.org/bitcoin.pdf']
    소스 :['https://github.com/bitcoin/']
    웹사이트 :['https://bitcoin.org/']
    설명 :Bitcoin (BTC) is a cryptocurrency . Users are able to generate BTC through the process of mining. Bitcoin has a current supply of 18,811,693. The last known price of Bitcoin is 46,614.98711064 USD and is up 0.94 over the last 24 hours. It is currently trading on 8840 active market(s) with $35,731,483,458.68 traded over the last 24 hours. More information can be found at https://bitcoin.org/.
    logo :https://s2.coinmarketcap.com/static/img/coins/64x64/1.png
    tags :['mineable', 'pow', 'sha-256', 'store-of-value', 'state-channels', 'coinbase-ventures-portfolio', 'three-arrows-capital-portfolio', 'polychain-capital-portfolio', 'binance-labs-portfolio', 'arrington-xrp-capital', 'blockchain-capital-portfolio', 'boostvc-portfolio', 'cms-holdings-portfolio', 'dcg-portfolio', 'dragonfly-capital-portfolio', 'electric-capital-portfolio', 'fabric-ventures-portfolio', 'framework-ventures', 'galaxy-digital-portfolio', 'huobi-capital', 'alameda-research-portfolio', 'a16z-portfolio', '1confirmation-portfolio', 'winklevoss-capital', 'usv-portfolio', 'placeholder-ventures-portfolio', 'pantera-capital-portfolio', 'multicoin-capital-portfolio', 'paradigm-xzy-screener']
  ```
🔔 **포스팅 공지** <br><br>
해당 포스팅은 제가 사용했던 코드를 정리하기 위한 용도로 설명이 조금 부실한점 양해 바랍니다 <br>
{: .notice--success}