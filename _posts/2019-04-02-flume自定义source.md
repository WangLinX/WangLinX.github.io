---
layout:     post
title:      flume 自定义 Source
subtitle:   flume 自定义 Source
date:       2019-04-02
author:     WLX
header-img:  
catalog: 	 true
tags:
    - flume
---

 - 自定义类

```java


package org.wlx.flume;

import org.apache.flume.Event;
import org.apache.flume.channel.ChannelProcessor;
import org.apache.flume.event.SimpleEvent;
import org.apache.flume.source.AbstractSource;

import java.util.HashMap;
import java.util.Map;

/**
 * 自定义Source
 */

public class mySource extends AbstractSource {

    public synchronized void start() {
        super.start();
        ChannelProcessor cp = this.getChannelProcessor();
        Map<String, String> map = new HashMap<String, String>();
        map.put("owner","wlx");
        map.put("data","2019/04/02");
        Event e = null;
        for(int i = 0; i <100000 ; i++){
            e = new SimpleEvent();
            e.setBody(("tom" + i).getBytes());
            e.setHeaders(map);
        }
        cp.processEvent(e);


    }
}

```

 - 导出到jar包
 
 右侧选择maven --- Lifecycle --- 右键 validate --- run maven build --- 成功后， 右键install --- run “project 名” \[install] 