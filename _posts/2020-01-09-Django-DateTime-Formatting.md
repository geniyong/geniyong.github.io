---
tags : Django
title : Datetime Formatting
---

```python
    from django.conf import settings
    from pytz import timezone

    korean_timezone = timezone(settings.TIME_ZONE)
    dt = datetime.datetime(2020, 1, 9, 11, 42, 7, 761239, tzinfo=<UTC>)
    korean_dt = dt.astimezone(korean_timezone)
    korean_dt.strftime("%Y-%m-%d %H:%M:%S")
    # return value : 2020-01-09 20:42:07
    # why hour value is 20 = 11 + 9(add_korean_time)
```