{% if site.data.keys %}
## GPG 公钥

{% for key in site.data.keys %}
<details>
    <summary>{{ key.name }}</summary>
    <blockquote>
        fingerprint: <span>{{ key.fingerprint }}</span><br/>
        download <a href="{{ key.file | absolute_url }}">.asc</a> file.
    </blockquote>
</details>
{% endfor %}
{% endif %}
