---
title: {{ title }}
date: {{ date }}
---

{% raw %}
<textarea id="flowchart_code" style="display: none;" rows="11">
<!-- 以下是流程图示例代码,可新增textarea，将id按照flowchart_code1，flowchart_code2...依次增加即可。注意html代码需要用{% raw %}...{% endraw %}括起来。
st=>start: Start|past:>http://www.google.com[blank]
e=>end: End:>http://www.google.com
op1=>operation: My Operation|past
op2=>operation: Stuff|current
sub1=>subroutine: My Subroutine|invalid
cond=>condition: Yes
or No?|approved:>http://www.google.com
c2=>condition: Good idea|rejected
io=>inputoutput: catch something...|request

st->op1(right)->cond
cond(yes, right)->c2
cond(no)->sub1(left)->op1
c2(yes)->io->e
c2(no)->op2->e
</textarea>
<div id="flowchart_canvas"></div>
-->
{% endraw %}
