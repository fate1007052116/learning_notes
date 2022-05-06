# JavaScript技巧积累

```javascript
// 使用vue+axios过程中，防止服务器响应之前，v-text/v-model就渲染空数据
var vue = new Vue({
    el:"#app",
    data:{
        content:{		// 手动写入空数据，响应到来的时候，直接将这个数据覆盖
            tbItem:""
        }
    },
    mounted(){
        var query = location.search.substring(1);
        var contentId = query.split("=")[1];
        console.log(query);

        axios.get("/content/item/getPanelContentByPanelContentId/"+contentId).then(
            function (resp) {
                vue.content = resp.data;
                console.log(vue.content);
            }
        )
    }
});
```

```html

这样使用v-bind 可以在axios数据渲染完成之后将input改为只读
<input type="text" v-model="content.tbItem.title"
	v-bind:readonly="{'true':content.tbItem.title}" onclick="chooseProduct()">
```

