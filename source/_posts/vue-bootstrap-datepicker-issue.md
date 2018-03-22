---
title: Vue.js中使用v-model无法取到bootstrap-datepicker的值
date: 2018-03-21 20:18:43
categories:
- keep learning
tags:
- Vue
- JQuery
- 前端

---

demo:

```html
    <div class="form-group">
                <label for="dtp_input1" class="col-md-2 control-label">DateTime Picking</label>
                <div class="input-group date form_datetime col-md-5" data-date="1979-09-16T05:25:07Z"
                     data-date-format="dd MM yyyy - HH:ii p" data-link-field="dtp_input1">
                    <input class="form-control" size="16" type="text" value="" readonly>
                    <span class="input-group-addon">
                        <span class="glyphicon glyphicon-remove"></span>
                    </span>
                    <span class="input-group-addon">
                        <span class="glyphicon glyphicon-th"></span>
                    </span>
                </div>
                <input type="hidden" id="dtp_input1" v-model="o.datetime" />
    </div>
```

o.datetime无法与dtp_input1绑定

----------
在vue github的issues中找到了原因：
>yyx990803 commented on 17 Nov 2016
>The fiddle isn't working, but it's most likely due to bootstrap-datepicker only triggering a jQuery change event instead of a real change event.

<https://github.com/vuejs/vue/issues/4231>

