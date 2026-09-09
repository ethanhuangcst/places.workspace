# Must-see nominate audit — 10 cities × CN/EN

Generated: 2026-09-08T01:42:51.105Z
Model: `qwen-plus`
Prompt: current `buildNominateMustSeeUserMessage` (trip prefs + day-trip + one cluster).
Fixed params: numDays=4, pace=medium, trip_type=城市漫游 / city wander (locale-matched).
Limit: 12 names; first 5 marked **[top5]**. LLM names only (not grounded search).
Not a product encyclopedia (ADR-042).

## 杭州 / Hangzhou

### 杭州 · CN

1. 西湖 **[top5]**
2. 灵隐寺 **[top5]**
3. 西溪湿地 **[top5]**
4. 河坊街 **[top5]**
5. 南宋御街 **[top5]**
6. 中国美术学院象山校区
7. 京杭大运河杭州段（拱宸桥一带）
8. 龙井村
9. 九溪烟树
10. 六和塔
11. 钱江新城灯光秀（城市阳台）
12. 千岛湖

### Hangzhou · EN

1. West Lake **[top5]**
2. Su Causeway **[top5]**
3. Bai Causeway **[top5]**
4. Leifeng Pagoda **[top5]**
5. Lingyin Temple **[top5]**
6. Xixi National Wetland Park
7. Hefang Street
8. China National Tea Museum (Longjing Branch)
9. Longjing Village
10. Wuzhen Water Town
11. Xitang Ancient Town
12. Qinghefang Ancient Street

## 厦门 / Xiamen

### 厦门 · CN

1. 鼓浪屿 **[top5]**
2. 南普陀寺 **[top5]**
3. 厦门大学 **[top5]**
4. 曾厝垵 **[top5]**
5. 中山路步行街 **[top5]**
6. 环岛路
7. 胡里山炮台
8. 沙坡尾艺术西区
9. 万石植物园
10. 集美学村
11. 海沧大桥观景台
12. 同安影视城

### Xiamen · EN

1. Gulangyu Island **[top5]**
2. Zhonghua Road Pedestrian Street **[top5]**
3. Nanputuo Temple **[top5]**
4. Xiamen University Campus **[top5]**
5. Hulishan Fortress **[top5]**
6. Shuzhuang Garden
7. Yundang Lake Park
8. Wanshi Botanical Garden
9. Zengcuo'an Village
10. Taiping Rock Park
11. Jimei University Town & Longchuan Bay
12. South Putuo Mountain Scenic Area

## 香港 / Hong Kong

### 香港 · CN

1. 维多利亚港 **[top5]**
2. 太平山顶 **[top5]**
3. 中环街市 **[top5]**
4. 南丫岛 **[top5]**
5. 大屿山昂坪360及天坛大佛 **[top5]**
6. 西九文化区
7. 深水埗街头艺术区
8. 黄大仙祠
9. 香港公园
10. 尖沙咀海滨长廊
11. 旺角女人街与金鱼街
12. 大坑舞火龙文化体验地

### Hong Kong · EN

1. Victoria Peak **[top5]**
2. Star Ferry **[top5]**
3. Temple Street Night Market **[top5]**
4. Lan Kwai Fong **[top5]**
5. Man Mo Temple **[top5]**
6. Hong Kong Park
7. Aberdeen Typhoon Shelter & Jumbo Kingdom (exterior view)
8. Tsim Sha Tsui Promenade
9. Wong Tai Sin Temple
10. Lantau Island (including Ngong Ping 360 and Po Lin Monastery)
11. Cheung Chau Island
12. Tai O Fishing Village

## 台北 / Taipei

### 台北 · CN

1. 故宮博物院 **[top5]**
2. 中正紀念堂 **[top5]**
3. 龍山寺 **[top5]**
4. 西門町 **[top5]**
5. 九份老街 **[top5]**
6. 淡水老街
7. 陽明山國家公園
8. 故宮南院
9. 華山1914文化創意產業園區
10. 松山文創園區
11. 剝皮寮歷史街區
12. 北投溫泉博物館

### Taipei · EN

1. Chiang Kai-shek Memorial Hall **[top5]**
2. Longshan Temple **[top5]**
3. National Palace Museum **[top5]**
4. Ximending **[top5]**
5. Shilin Night Market **[top5]**
6. Taipei 101 Observatory
7. Yangmingshan National Park
8. Beitou Hot Springs
9. Jiufen Old Street
10. Shifen Waterfall and Sky Lanterns
11. Yehliu Geopark
12. Tamsui Fisherman's Wharf

## 里斯本 / Lisbon

### 里斯本 · CN

1. 贝伦塔 **[top5]**
2. 热罗尼姆斯修道院 **[top5]**
3. 阿尔法玛老城区 **[top5]**
4. 圣乔治城堡 **[top5]**
5. 商业广场 **[top5]**
6. 四月二十五日大桥
7. 辛特拉宫
8. 罗卡角
9. 里斯本电车28路沿线
10. 国家瓷砖博物馆
11. 卡尔莫修道院废墟
12. 蒙萨拉什观景台

### Lisbon · EN

1. Alfama **[top5]**
2. Belém Tower **[top5]**
3. Jerónimos Monastery **[top5]**
4. Castelo de São Jorge **[top5]**
5. Tram 28 Route **[top5]**
6. Praça do Comércio
7. Bairro Alto
8. Miradouro de Santa Luzia
9. Parque das Nações
10. Sintra (Palácio Nacional de Sintra & Quinta da Regaleira)
11. Cascais
12. Caboo do Roca

## 上海 / Shanghai

### 上海 · CN

1. 外滩 **[top5]**
2. 豫园 **[top5]**
3. 南京路步行街 **[top5]**
4. 上海中心大厦观光厅 **[top5]**
5. 武康路-安福路历史街区 **[top5]**
6. 田子坊
7. 朱家角古镇
8. 西塘古镇
9. 中共一大会址
10. 上海博物馆
11. 徐家汇源
12. 苏州河步道（昌平路桥至浙江路桥段）

### Shanghai · EN

1. The Bund **[top5]**
2. Yu Garden **[top5]**
3. Xintiandi **[top5]**
4. Nanjing Road Pedestrian Street **[top5]**
5. French Concession (including Fuxing Park and Wukang Road) **[top5]**
6. Shanghai Tower
7. Zhujiajiao Water Town
8. Tianzifang
9. Jade Buddha Temple
10. West Bund (including Long Museum and滨江步道)
11. Huangpu River Cruise
12. Shanghai Museum

## 西安 / Xi'an

### 西安 · CN

1. 兵马俑 **[top5]**
2. 大雁塔 **[top5]**
3. 西安城墙 **[top5]**
4. 钟楼 **[top5]**
5. 鼓楼 **[top5]**
6. 回民街
7. 陕西历史博物馆
8. 大唐不夜城
9. 华清宫
10. 骊山
11. 小雁塔
12. 大明宫国家遗址公园

### Xi'an · EN

1. Terracotta Army **[top5]**
2. Xi'an City Wall **[top5]**
3. Bell Tower **[top5]**
4. Drum Tower **[top5]**
5. Muslim Quarter **[top5]**
6. Big Wild Goose Pagoda
7. Small Wild Goose Pagoda
8. Shaanxi History Museum
9. Huaqing Pool
10. Mount Li
11. Famen Temple
12. Qin Shi Huang's Mausoleum Site Museum

## 伦敦 / London

### 伦敦 · CN

1. 大本钟 **[top5]**
2. 伦敦眼 **[top5]**
3. 白金汉宫 **[top5]**
4. 大英博物馆 **[top5]**
5. 威斯敏斯特教堂 **[top5]**
6. 伦敦塔
7. 圣保罗大教堂
8. 特拉法加广场
9. 考文特花园
10. 海德公园
11. 泰特现代美术馆
12. 温莎城堡

### London · EN

1. Buckingham Palace **[top5]**
2. Westminster Abbey **[top5]**
3. The Houses of Parliament and Big Ben **[top5]**
4. The Tower of London **[top5]**
5. British Museum **[top5]**
6. National Gallery
7. St Paul's Cathedral
8. Trafalgar Square
9. Hyde Park
10. South Bank (including Shakespeare's Globe and Tate Modern)
11. Camden Market
12. Stonehenge

## 新加坡 / Singapore

### 新加坡 · CN

1. 滨海湾花园 **[top5]**
2. 鱼尾狮公园 **[top5]**
3. 牛车水 **[top5]**
4. 小印度 **[top5]**
5. 甘榜格南 **[top5]**
6. 新加坡国家博物馆
7. 植物园
8. 圣淘沙岛（西罗索海滩与海事博物馆）
9. 克拉码头
10. 乌节路
11. 麦里芝蓄水池
12. 裕廊湖花园

### Singapore · EN

1. Marina Bay Sands **[top5]**
2. Gardens by the Bay **[top5]**
3. Sentosa Island **[top5]**
4. Chinatown **[top5]**
5. Little India **[top5]**
6. Kampong Glam
7. Singapore Botanic Gardens
8. Haw Par Villa
9. Jurong Lake Gardens
10. MacRitchie Reservoir
11. Pulau Ubin
12. Southern Ridges

## 清迈 / Chiang Mai

### 清迈 · CN

1. 清迈古城 **[top5]**
2. 素贴山双龙寺 **[top5]**
3. 清迈夜间动物园 **[top5]**
4. 清迈周末夜市 **[top5]**
5. 宁曼路 **[top5]**
6. 清迈大学
7. 塔佩门
8. 契迪龙寺
9. 帕辛寺
10. 清迈兰花园
11. 湄登大象自然公园
12. 拜县

### Chiang Mai · EN

1. Wat Phra Singh **[top5]**
2. Wat Chedi Luang **[top5]**
3. Doi Suthep-Pui National Park **[top5]**
4. Sunday Walking Street (Tha Phae Road) **[top5]**
5. Wat Phra That Doi Kham **[top5]**
6. Huay Tung Tao Lake & Temple
7. Mae Sa Valley (including Mae Sa Waterfall and Elephant Nature Park)
8. Chiang Mai Night Bazaar
9. Wat Umong
10. Suthep Mountain Viewpoint (near Wat Phra That Doi Suthep)
11. Lanna Architecture District (Ratchadamnoen Road & surrounding historic lanes)
12. Baan Kang Wat Artist Village
