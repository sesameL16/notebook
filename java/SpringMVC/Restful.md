​		如果@RequestMapping中表示为/itemsView/{id}，id和形参名称一致，@PathVariable不用指定名称。

~~~java
@RequestMapping("/itemsView/{id}")
@ResponseBody
public ItemsCustom itemsView(@PathVariable("id") Integer items_id)throws Exception{
	//调用service查询商品信息
    ItemsCustom itemsCustom = itemsService.findItemsById(items_id);
	return itemsCustom;
}
~~~

