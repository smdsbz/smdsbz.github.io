---
layout: article
title: 你 B 直播弹幕 WebSocket 格式解包
key: bilibili-live-danmaku-websocket-packet-format
tags: Blivechat Bilibili WebSocket
---

Reverse engineering non-commented JSON.

<!-- more -->


```json
// 表情弹幕
{
    "cmd": "DANMU_MSG",
    "info": [
        [
            0,
            1,
            25,
            16777215,
            1668345037096,
            912018007,
            0,
            "1dd989cf",
            0,
            0,
            0,
            "",
            1,
            {               // ['info'][0][13]，表情弹幕
                "bulge_display": 0,
                "emoticon_unique": "official_120",
                "height": 60,
                "in_player_area": 1,
                "is_dynamic": 1,
                "url": "http://i0.hdslb.com/bfs/live/9029486931c3169c3b4f8e69da7589d29a8eadaa.png",
                "width": 159
            },
            "{}",
            {
                "mode": 0,
                "show_player_type": 0,
                "extra": "{\"send_from_me\":false,\"mode\":0,\"color\":16777215,\"dm_type\":1,\"font_size\":25,\"player_mode\":1,\"show_player_type\":0,\"content\":\"离谱\",\"user_hash\":\"500795855\",\"emoticon_unique\":\"official_120\",\"bulge_display\":0,\"recommend_score\":2,\"main_state_dm_color\":\"\",\"objective_state_dm_color\":\"\",\"direction\":0,\"pk_direction\":0,\"quartet_direction\":0,\"anniversary_crowd\":0,\"yeah_space_type\":\"\",\"yeah_space_url\":\"\",\"jump_to_url\":\"\",\"space_type\":\"\",\"space_url\":\"\"}"
            },
            {
                "activity_identity": "",
                "activity_source": 0,
                "not_show": 0
            }
        ],
        "离谱",
        [
            1741226472,
            "归途的疾风来了",
            0,
            0,
            0,
            10000,
            1,
            ""
        ],
        [],
        [
            0,
            0,
            9868950,
            ">50000",
            0
        ],
        [
            "",
            ""
        ],
        0,
        0,
        null,
        {
            "ts": 1668345037,
            "ct": "3CE61D6A"
        },
        0,
        0,
        null,
        null,
        0,
        7
    ]
}



//普通弹幕
{
    "cmd": "DANMU_MSG",
    "info": [
        [
            0,
            1,
            25,
            16772431,
            1668345023584,
            1668342446,
            0,
            "6a2851f3",
            0,
            0,
            5,
            "#1453BAFF,#4C2263A2,#3353BAFF",
            0,
            "{}",
            "{}",
            {
                "mode": 0,
                "show_player_type": 0,
                "extra": "{\"send_from_me\":false,\"mode\":0,\"color\":16772431,\"dm_type\":0,\"font_size\":25,\"player_mode\":1,\"show_player_type\":0,\"content\":\"果然淡淡哥想要13140\",\"user_hash\":\"1781027315\",\"emoticon_unique\":\"\",\"bulge_display\":0,\"recommend_score\":1,\"main_state_dm_color\":\"\",\"objective_state_dm_color\":\"\",\"direction\":0,\"pk_direction\":0,\"quartet_direction\":0,\"anniversary_crowd\":0,\"yeah_space_type\":\"\",\"yeah_space_url\":\"\",\"jump_to_url\":\"\",\"space_type\":\"\",\"space_url\":\"\"}"
            },
            {
                "activity_identity": "",
                "activity_source": 0,
                "not_show": 0
            }
        ],
        "果然淡淡哥想要13140",
        [
            513413059,
            "bili_誓约",
            1,
            0,
            0,
            10000,
            1,
            "#00D1F1"
        ],
        [
            25,
            "扯尾巴",
            "我给尾巴找土拨鼠",
            25666818,
            398668,
            "",
            0,
            6809855,
            398668,
            6850801,
            3,
            1,
            1279251805
        ],
        [
            20,
            0,
            6406234,
            ">50000",
            0
        ],
        [
            "",
            ""
        ],
        0,
        3,
        null,
        {
            "ts": 1668345023,
            "ct": "D59F8342"
        },
        0,
        0,
        null,
        null,
        0,
        105
    ]
}
```
