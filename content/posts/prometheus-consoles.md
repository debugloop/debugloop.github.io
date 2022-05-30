---
title: "Prometheus Consoles and curl"
date: 2020-04-11T22:16:48+02:00
tags:
- prometheus
- monitoring
summary: |
  An apparently little used feature in Prometheus is the ability to create
  so-called consoles or console templates. Even though Prometheus ships
  some example consoles, this type of interface seems to be rarely used in
  practice.
---

An apparently little used feature in [Prometheus][0] is the ability to create
so-called consoles or [console templates][1]. Even though Prometheus ships
some example consoles, this type of interface seems to be rarely used in
practice. This is probably due to the reason that [Grafana][2] provides an
excellent and polished interface for all visialization needs, while Prometheus'
builtin web interface covers most administrative tasks and works well in all
gimme-raw-metrics-right-quick-style scenarios.

I've however found the flexibility the consoles interface offers working well
for specialized views into your data. In my opinion, console templates and
especially the dashboard-style alternatives shipped by Prometheus are not an
adequate Grafana replacement. However, creating custom console templates is
entirely possible and offers nearly infinite possibilities.

## Command Line Interfaces with curl

One application of consoles is the ability to create interfaces intended for
[curl][3] instead of a regular browser. This is a well-known concept, there is
for example the popular and `curl`-enabled [wttr.in][4]. Applying this concept
to Prometheus consoles, `curl
"http://hostname:9090/consoles/show_int?hostname=fra-decix-1"` can yield this:

{{<figure src="/images/prometheus-consoles/curl.png" link="/prometheus-consoles/curl.png" alt="Console Template curl">}}

The code required to make this work is the [Golang template][5] at the end of
this paragraph. These templates can get quite messy and tend to contain some
amount of boilerplate code. The magic happens in the highlighted lines, which
take a HTTP GET param and use that to build Prometheus queries, which in turn
is being iterated over from line 20 on. This example relies on all three series
being returned in the same order, but one can easily adapt this to look up
corresponding values in case the stored time series are less homogenous.

```go {hl_lines=[7,8,9]}
{{- /* ####### templates for console colors and formatting */ -}}
{{- define "blue_bps" -}}{{ printf "\033[1;34m%7sbps\033[0m" . }}{{- end -}}
{{- define "red" -}}{{ printf "\033[1;31m%4s\033[0m" . }}{{- end -}}
{{- define "green" -}}{{ printf "\033[1;32m%4s\033[0m" . }}{{- end -}}

{{- /* ####### query data from prometheus */ -}}
{{- $state := printf "interface_state{hostname='%s'}" .Params.hostname | query -}}
{{- $in_samples := printf "interface_in_bps{hostname='%s'}" .Params.hostname | query -}}
{{- $out_samples := printf "interface_out_bps{hostname='%s'}" .Params.hostname | query -}}

{{- /* ####### print header */ -}}
{{ print "STAT |  IN BPS  |  OUT BPS |           NAME          | DESC\n" }}

{{- /* ####### for each returned interface... */ -}}
{{- range $i, $iface := $state -}}

{{- /* ####### write the first column as a string based on the value, in color */ -}}
{{- if eq ($iface | value) 1.0 -}}
{{ template "green" "UP" }} |
{{- else if ($iface | value) 2.0 -}}
{{ template "red" "DOWN" }} |
{{- else -}}
{{ template "red" "UNKN" }} |
{{- end -}}

{{- /* ####### write the in bps column */ -}}
{{- template "blue_bps" index $in_samples $i | value | humanize -}} |
{{- template "blue_bps" index $out_samples $i | value | humanize -}} |
{{- printf " %24s" ($iface | label "name") -}} |
{{- $iface | label "description" }}
{{ end -}}
```

## What to do with this?
I'm not sure yet. While I've dubbed above example `show_int` after the common
router command (which lacks the current bits per second of course), it's
unclear where to go from here.

Using this interface for machine readable output does not seem wise, although
one could in theory output the [InfluxDB line protocol][7], for instance.
However, InfluxDB can use Prometheus' remote read API endpoint directly.

[0]: https://prometheus.io
[1]: https://prometheus.io/docs/visualization/consoles/
[2]: https://grafana.com/
[3]: https://curl.haxx.se/
[4]: https://wttr.in/
[5]: https://golang.org/pkg/text/template/
[6]: https://icanhazip.com/
[7]: https://v2.docs.influxdata.com/v2.0/reference/syntax/line-protocol/
