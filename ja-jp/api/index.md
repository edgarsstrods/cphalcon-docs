---
layout: default
title: 'API インデックス'
---

## API インデックス
{% for element in site.data.api %}
### {{ element['title'] }}

    {% for document in element['docs'] %}
* [{{ document | replace: '_', '\' }}](/{{ pageVersion }}/{{ pageLanguage }}/api/{{ element['title'] | replace: '\', '_' | downcase }}#{{ document | replace: '/', '-' | downcase }})
    {% endfor %}
{% endfor %}
