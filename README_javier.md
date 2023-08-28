## for major features
### for sub features


## Major feature
## Sub feature
file pathway for subfeature
code for subfeature
subfeature section explanation


## GetX used for drop-down list of news articles

### RxList for list collecion of news articles
lib/pages/category/state.dart
```dart
class CategoryState {
  // 新闻翻页
  RxList<NewsItem> newsList = <NewsItem>[].obs;
}
```

### Pull_to_refresh function for refresh feature of news articles list
lib/pages/category/widgets/news_page_list.dart
```dart
 @override
  Widget build(BuildContext context) {
    super.build(context);
    return GetX<CategoryController>(
      init: controller,
      builder: (controller) => SmartRefresher(
        enablePullUp: true,
        controller: controller.refreshController,
        onRefresh: controller.onRefresh,
        onLoading: controller.onLoading,
        child: CustomScrollView(
          slivers: [
            SliverPadding(
              padding: EdgeInsets.symmetric(
                vertical: 0.w,
                horizontal: 0.w,
              ),
              sliver: SliverList(
                delegate: SliverChildBuilderDelegate(
                  (content, index) {
                    var item = controller.state.newsList[index];
                    return newsListItem(item);
                  },
                  childCount: controller.state.newsList.length,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
```

`controller: controller.refreshController` : Pull-down controller
`onRefresh: controller.onRefresh`: Refresh data upon pulling down smartrefresher widget
`onLoading: controller.onLoading`: Load data upon pulling up smartrefresher widget
`SliverChildBuilderDelegate`: Dynamically build sliver child item and uses 'childCount' retrieved from CategoryController to tell 'SliverList'  
how many items there are in total

### CategoryController writing items
lib/pages/category/controller.dart
- 'onRefresh' Refresh on pull-down
```dart
  void onRefresh() {
    fetchNewsList(isRefresh: true).then((_) {
      refreshController.refreshCompleted(resetFooterState: true);
    }).catchError((_) {
      refreshController.refreshFailed();
    });
  }
```
'refreshController.refreshCompleted()' refresh completed
'refreshController.refreshFailed()' Failed to refresh

-'onLoading' pull up loading 
```dart
  void onLoading() {
    if (state.newsList.length < total) {
      fetchNewsList().then((_) {
        refreshController.loadComplete();
      }).catchError((_) {
        refreshController.loadFailed();
      });
    } else {
      refreshController.loadNoData();
    }
  }
```
`refreshController.loadComplete()` Loading completed
`refreshController.loadFailed()` Loading failed
`refreshController.laodNoData()` No new data to be loaded 

- Fetching all data
```dart
  // 拉取数据
  Future<void> fetchNewsList({bool isRefresh = false}) async {
    var result = await NewsAPI.newsPageList(
      params: NewsPageListRequestEntity(
        categoryCode: categoryCode,
        pageNum: curPage + 1,
        pageSize: pageSize,
      ),
    );

    if (isRefresh == true) {
      curPage = 1;
      total = result.counts!;
      state.newsList.clear();
    } else {
      curPage++;
    }

    state.newsList.addAll(result.items!);
  }
```