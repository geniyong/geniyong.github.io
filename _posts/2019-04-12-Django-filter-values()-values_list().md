# Django values() vs values_list()
## Django 에서 values() 와 values_list() 의 차이점

``` python
Article.objects.values('comment_id').distinct()
```

와

``` python
Article.objects.values_list('comment_id', flat=True).distinct()
```

## 쿼리 결과
`values()` 메서드는 `딕셔너리`를 가지고 있는 `쿼리셋`을 리턴한다
``` python
<QuerySet [{'comment_id': 1}, {'comment_id': 2}]>
```

`values_list()` 메서드는 `튜플`을 가지고 있는 `쿼리셋`을 리턴한다
``` python
<QuerySet [(1,), (2,)]>
```

`values_list()` 메서드에서 `flat=True` 옵션을 사용하면 튜플이 아니라 `단일 값`의 `쿼리셋`을 리턴한다
``` python
<QuerySet [1, 2]>
```