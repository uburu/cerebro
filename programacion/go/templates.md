<https://golang.org/pkg/text/template/>
<https://golang.org/pkg/html/template/>

Templating online usando un JSON como variables:
<https://gotemplate.io/>

<http://masterminds.github.io/sprig/>
Para tener más funciones en los templates, por ejemplo, default, upper, etc
{{ .VRID | default 40 }}

<https://github.com/subfuzion/envtpl>
cli para renderizar templates usando variables de entorno
mirar virtualizacion/docker/config.md

ejemplos:
<http://play.golang.org/p/vVM89Ze40U>
<https://jan.newmarch.name/go/template/chapter-template.html>

  tpl, err := template.ParseFiles("templates/template.html")
  if err != nil {
    panic(err)
  }

  type Page struct {
    Title string
    Body  []byte
  }

  data := &Page{Title: "title"}

  body := new(bytes.Buffer)
  err = tpl.Execute(email_body, data)
  if err != nil {
    panic(err)
  }

  fmt.Println(body.String())

Variables:
{{.Title}}

{{$iface := .IFACE }}
{{$vips := splitList "," .VIP }}
  función de sprig

Condicionales:
{{if .C}}
  /path/
{{else}}
  /search?q=
{{end}}

{{ if eq .State "critical" }}
...

Else-If
{{if eq .Active "accueil"}}
class="active"
{{else if eq .Active "pepe"}}
PEPE
{{else}}
no pepe no accueil
{{end}}

# Hacer cosas si la variable es o no es empty

Solo sacar linea si tenemos elemento (en . pondra .Gift):
{{with .Gift}}Thank you for the lovely {{.}}{{ else }}Otra cosa{{ end }}

Slice vacia?
{{ if ne 0 (len .Roles) }}
tiene elementos
{{ else }}
slice vacia
{{ end }}

Agregar otros templates:
{{template "pepe"}}

# Asignar variables

{{ $addrLen := len $value.Addresses }}

# Range / loop / bucles

{{ range $key, $value := . }}
 <li><strong>{{ $key }}</strong>: {{ $value }}</li>
{{ end }}

Si queremos acceder a una variable "global" (definida fuera del range): $.FOO

{{range .Mounts}}{{.Source}} -> {{.Destination}}{{end}}

# Elemento de un slice

index .spec.ports 0
sería algo así como spec.ports[0]

# Eliminar espacios en blanco / newlines

{{- end}}

# Mostrar todas las variables disponibles

{{ printf "%+v" . }}
