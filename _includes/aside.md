<!--page footer -->
<div class="aside">
<div class="me">
        <div class="title">ABOUT ME</div>
        <div class="info">[码农]</div>
</div>


<div class="links">
        {% for i in site.customLinks %}
        <h4 class="links-category fn-clear">{{ i.message }}</h4>
        <ul class="links-list fn-clear">
                {% for link in i.item %}
                <li><a href="{{ link.url }}" target="_blank">{{ link.text }}</a></li>
                {% endfor %}
        </ul>
        {% endfor %}
</div>

</div>


