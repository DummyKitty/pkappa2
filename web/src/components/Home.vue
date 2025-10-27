<template>
  <v-card>
    <v-card-title>帮助</v-card-title>
    <v-card-text>
      本页提供可用过滤器的快速概览，以及如何用它们搜索数据流。你可以使用顶部搜索栏输入查询，并借助过滤器收窄结果范围。
      <br />
      详细教程请参考
      <a
        href="https://github.com/spq/pkappa2?tab=readme-ov-file#searching-streams"
        target="_blank"
        rel="noopener"
        >README</a
      >。
    </v-card-text>
  </v-card>
  <v-card>
    <v-card-title>快捷键</v-card-title>
    <v-card-text>
      <v-table class="help-table">
        <colgroup>
          <col style="width: 220px" />
          <col style="width: 220px" />
          <col />
        </colgroup>
        <tbody>
          <tr>
            <th>搜索</th>
            <td><code>/</code></td>
            <td>聚焦到搜索输入框。</td>
          </tr>
          <tr>
            <th>滚动查询历史</th>
            <td><code>Arrow up / down</code></td>
            <td>
              当焦点在搜索输入框时，滚动浏览查询历史。
            </td>
          </tr>
          <tr>
            <th>上一条结果</th>
            <td><code>j</code></td>
            <td>
              在结果列表：上一页。流详情页：跳到结果列表中的上一条流。
            </td>
          </tr>
          <tr>
            <th>下一条结果</th>
            <td><code>k</code></td>
            <td>
              在结果列表：下一页。流详情页：跳到结果列表中的下一条流。
            </td>
          </tr>
          <tr>
            <th>多选标签</th>
            <td><code>Shift + 点击侧边栏标签</code></td>
            <td>在查询中添加/移除多个服务或标签。</td>
          </tr>
        </tbody>
      </v-table>
    </v-card-text>
  </v-card>
  <v-card>
    <v-card-title>查询语法</v-card-title>
    <v-card-text>
      <v-table class="help-table">
        <colgroup>
          <col style="width: 220px" />
          <col style="width: 320px" />
          <col />
        </colgroup>
        <tbody>
          <tr>
            <th>运算符</th>
            <td><code>filter&nbsp;[AND]|OR|THEN&nbsp;filter</code></td>
            <td width="100%">
              <code>AND</code>/<code>OR</code> 表示与/或。
              <code>THEN</code> 类似 <code>AND</code>，但要求 <code>[cs]data</code> 过滤器按顺序匹配。
              <code>AND</code> 可省略。
            </td>
          </tr>
          <tr>
            <th>括号</th>
            <td><code>(filter)</code></td>
            <td width="100%">可用括号对过滤器分组。</td>
          </tr>
          <tr>
            <th>取反</th>
            <td><code>-filter</code></td>
            <td width="100%">对过滤条件取反。</td>
          </tr>
          <tr>
            <th>过滤器格式</th>
            <td>
              <code>key:value</code>&nbsp;或&nbsp;<code>key:"value"</code>
            </td>
            <td width="100%">
              不含空格、引号、括号等特殊字符时可用前者，否则使用带引号格式；引号通过重复来转义。
            </td>
          </tr>
          <tr>
            <th>子查询</th>
            <td><code>@name:id:123</code></td>
            <td width="100%">
              在任意过滤器前添加 <code>@子查询名:</code> 即可引用子查询。
            </td>
          </tr>
          <tr>
            <th>变量</th>
            <td><code>@id@</code> 或 <code>@subquery:ftime@</code></td>
            <td width="100%">
              多数过滤器支持变量。若引用其他子查询中的变量，需加上子查询名和 <code>:</code> 前缀。
            </td>
          </tr>
          <tr>
            <th>标签/服务/标记/生成 过滤器</th>
            <td>
              <code>tag:tagname,othertag</code>、<code>service:svc</code>、
              <code>mark:marked</code> 或 <code>generated:foo</code>
            </td>
            <td width="100%">
              将结果限制为匹配指定 标签/服务/生成/标记 的流；多个名称用 <code>,</code> 分隔。
            </td>
          </tr>
          <tr>
            <th>协议过滤器</th>
            <td><code>protocol:tcp,udp</code></td>
            <td width="100%">
              仅显示指定协议（<code>tcp</code>、<code>udp</code>、<code>sctp</code>）的流，多个协议用 <code>,</code> 分隔；支持变量（如 <code>protocol:@subquery:protocol@</code>）。
            </td>
          </tr>
          <tr>
            <th>ID 过滤器</th>
            <td><code>id:1,2,3,@subquery:id@+123</code></td>
            <td width="100%">
              仅显示给定 ID 的流；可提供以 <code>,</code> 分隔的 ID 列表，或使用 <code>:</code> 表示的区间（任一端可留空表示开区间）。
              可使用（含子查询的）变量 <code>id</code>、<code>[cs]port</code>、<code>[cs]bytes</code>，并支持 <code>+</code>/<code>-</code> 简单计算。
            </td>
          </tr>
          <tr>
            <th>端口过滤器</th>
            <td><code>[cs]port:80,1024:,</code></td>
            <td width="100%">
              <code>cport</code>（客户端）、<code>sport</code>（服务端）或 <code>port</code>（任意）按端口过滤，语法与 <code>id</code> 过滤器一致。
            </td>
          </tr>
          <tr>
            <th>字节数过滤器</th>
            <td><code>[cs]bytes:1337,2048:4096</code></td>
            <td width="100%">
              <code>cbytes</code>、<code>sbytes</code>、<code>bytes</code> 按客户端/服务端/任意方向发送的字节数过滤，语法与 <code>id</code> 相同。
            </td>
          </tr>
          <tr>
            <th>主机过滤器</th>
            <td>
              <code>[cs]host:1.2.3.4,10.0.0.0/8,::1,10.0.0.1/8/-8</code>
            </td>
            <td width="100%">
              <code>chost</code>、<code>shost</code>、<code>host</code> 分别按客户端/服务端/任意主机过滤；支持列表。每一项可以是 IP 或变量（如 <code>@subquery:[cs]host@</code>）。可附加一个或多个 <code>/bits</code> 后缀，支持负数位，例如 <code>/16/-8</code>。
            </td>
          </tr>
          <tr>
            <th>时间过滤器</th>
            <td>
              <code>[fl]time:-1h:,1300:1400,@subquery:ftime@-5m:</code>
            </td>
            <td width="100%">
              按首包（<code>ftime</code>）、末包（<code>ltime</code>）或任意包（<code>time</code>）所在时间范围过滤。支持列表与开区间；区间端点可为相对时间（如 <code>-2h3m4s</code>）或绝对时间（格式 <code>[YYYY-MM-DD ]HHMM[SS]</code>）。支持 <code>[fl]time</code> 变量及 <code>+</code>/<code>-</code> 计算，例如查找持续 ≥5 分钟可用 <code>ltime:@ftime@+5m:</code>。
            </td>
          </tr>
          <tr>
            <th>数据过滤器</th>
            <td><code>[cs]data[.converter]:flag[{}].+[}]</code></td>
            <td width="100%">
              在客户端（<code>cdata</code>）、服务端（<code>sdata</code>）或任意方向（<code>data</code>）的数据中匹配正则。正则语法参考
              <a
                href="https://golang.org/pkg/regexp/syntax/#hdr-Syntax"
                target="_blank"
                >Golang 正则语法</a
              >。在同一组由 <code>then</code> 连接的 data 过滤器中，可引用之前命名捕获组生成的变量；也可在 <code>and</code> 连接的场景中引用其他子查询变量。可选在 <code>.</code> 后指定转换器名；省略则对原始数据及所有附加转换器输出同时匹配；设置为 <code>none</code> 则仅对原始数据匹配。
            </td>
          </tr>
          <tr>
            <th>排序</th>
            <td><code>sort:saddr,ftime,-id</code></td>
            <td width="100%">
              使用 <code>sort</code> 可以为结果排序（查询中仅出现一次）。值为以 <code>,</code> 分隔的字段列表，字段前加 <code>-</code> 表示倒序。可用字段：<code>id</code>、<code>[fl]time</code>、<code>[cs]bytes</code>、<code>[cs]host</code>、<code>[cs]port</code>；默认 <code>-ftime</code>。
            </td>
          </tr>
          <tr>
            <th>限制结果数量</th>
            <td><code>limit:10</code></td>
            <td width="100%">
              <code>limit</code> 限制返回的结果数量（仅数字）。默认 <code>100</code>，<code>0</code> 表示不限制。
            </td>
          </tr>
          <tr>
            <th>分组</th>
            <td><code>group:"@sport@"</code></td>
            <td width="100%">
              按参数中的变量对结果分组。目前不支持子查询变量。
            </td>
          </tr>
        </tbody>
      </v-table>
    </v-card-text>
  </v-card>
</template>

<script lang="ts" setup></script>

<style scoped>
.help-table :deep(table) {
  table-layout: fixed;
}
.help-table :deep(th),
.help-table :deep(td) {
  vertical-align: top;
  word-break: break-word;
}
.help-table :deep(code) {
  white-space: pre-wrap;
}
</style>
