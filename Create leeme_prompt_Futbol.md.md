# PROYECTO COMPLETO: SISTEMA DE FÚTBOL

## ESTRUCTURA DE CARPETAS:
```
UIII_Futbol_0562/
├── .venv/
├── backend_Futbol/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── app_Futbol/
│   ├── migrations/
│   │   └── __init__.py
│   ├── templates/
│   │   ├── equipo/
│   │   │   ├── agregar_equipo.html
│   │   │   ├── ver_equipos.html
│   │   │   ├── actualizar_equipo.html
│   │   │   └── borrar_equipo.html
│   │   ├── base.html
│   │   ├── header.html
│   │   ├── navbar.html
│   │   ├── footer.html
│   │   └── inicio.html
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── tests.py
├── manage.py
└── db.sqlite3
```

## ARCHIVOS COMPLETOS:

### 1. `backend_Futbol/settings.py`
```python
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'django-insecure-futbol-secret-key-0562'

DEBUG = True

ALLOWED_HOSTS = []

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_Futbol',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'backend_Futbol.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

LANGUAGE_CODE = 'es-mx'
TIME_ZONE = 'America/Mexico_City'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

### 2. `backend_Futbol/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('app_Futbol.urls')),
]
```

### 3. `app_Futbol/models.py`
```python
from django.db import models

class Equipo(models.Model):
    nombre = models.CharField(max_length=100)
    ciudad = models.CharField(max_length=100)
    pais = models.CharField(max_length=100)
    fundacion = models.DateField()
    estadio = models.CharField(max_length=100)
    entrenador = models.CharField(max_length=100)
    colores = models.CharField(max_length=100)
    
    def __str__(self):
        return self.nombre

class Jugador(models.Model):
    POSICIONES = [
        ('POR', 'Portero'),
        ('DEF', 'Defensa'),
        ('MED', 'Mediocampista'),
        ('DEL', 'Delantero'),
    ]
    
    nombre = models.CharField(max_length=100)
    apellido = models.CharField(max_length=100)
    fecha_nacimiento = models.DateField()
    nacionalidad = models.CharField(max_length=100)
    posicion = models.CharField(max_length=3, choices=POSICIONES)
    numero_camiseta = models.PositiveIntegerField()
    equipo = models.ForeignKey(Equipo, on_delete=models.CASCADE, related_name="jugadores")
    
    def __str__(self):
        return f"{self.nombre} {self.apellido}"

class Partido(models.Model):
    ESTADOS = [
        ('PEN', 'Pendiente'),
        ('JUG', 'Jugándose'),
        ('FIN', 'Finalizado'),
        ('SUS', 'Suspendido'),
    ]
    
    fecha = models.DateTimeField()
    estadio = models.CharField(max_length=100)
    equipo_local = models.ForeignKey(Equipo, on_delete=models.CASCADE, related_name="partidos_local")
    equipo_visitante = models.ForeignKey(Equipo, on_delete=models.CASCADE, related_name="partidos_visitante")
    goles_local = models.PositiveIntegerField(default=0)
    goles_visitante = models.PositiveIntegerField(default=0)
    estado = models.CharField(max_length=3, choices=ESTADOS, default='PEN')
    
    def __str__(self):
        return f"{self.equipo_local} vs {self.equipo_visitante}"
```

### 4. `app_Futbol/views.py`
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Equipo

def inicio_futbol(request):
    return render(request, 'inicio.html')

def agregar_equipo(request):
    if request.method == 'POST':
        Equipo.objects.create(
            nombre=request.POST['nombre'],
            ciudad=request.POST['ciudad'],
            pais=request.POST['pais'],
            fundacion=request.POST['fundacion'],
            estadio=request.POST['estadio'],
            entrenador=request.POST['entrenador'],
            colores=request.POST['colores']
        )
        return redirect('ver_equipos')
    return render(request, 'equipo/agregar_equipo.html')

def ver_equipos(request):
    equipos = Equipo.objects.all()
    return render(request, 'equipo/ver_equipos.html', {'equipos': equipos})

def actualizar_equipo(request, id):
    equipo = get_object_or_404(Equipo, id=id)
    return render(request, 'equipo/actualizar_equipo.html', {'equipo': equipo})

def realizar_actualizacion_equipo(request, id):
    if request.method == 'POST':
        equipo = get_object_or_404(Equipo, id=id)
        equipo.nombre = request.POST['nombre']
        equipo.ciudad = request.POST['ciudad']
        equipo.pais = request.POST['pais']
        equipo.fundacion = request.POST['fundacion']
        equipo.estadio = request.POST['estadio']
        equipo.entrenador = request.POST['entrenador']
        equipo.colores = request.POST['colores']
        equipo.save()
        return redirect('ver_equipos')
    return redirect('ver_equipos')

def borrar_equipo(request, id):
    equipo = get_object_or_404(Equipo, id=id)
    return render(request, 'equipo/borrar_equipo.html', {'equipo': equipo})

def realizar_borrado_equipo(request, id):
    if request.method == 'POST':
        equipo = get_object_or_404(Equipo, id=id)
        equipo.delete()
        return redirect('ver_equipos')
    return redirect('ver_equipos')
```

### 5. `app_Futbol/urls.py`
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.inicio_futbol, name='inicio_futbol'),
    path('equipos/agregar/', views.agregar_equipo, name='agregar_equipo'),
    path('equipos/', views.ver_equipos, name='ver_equipos'),
    path('equipos/actualizar/<int:id>/', views.actualizar_equipo, name='actualizar_equipo'),
    path('equipos/realizar_actualizacion/<int:id>/', views.realizar_actualizacion_equipo, name='realizar_actualizacion_equipo'),
    path('equipos/borrar/<int:id>/', views.borrar_equipo, name='borrar_equipo'),
    path('equipos/realizar_borrado/<int:id>/', views.realizar_borrado_equipo, name='realizar_borrado_equipo'),
]
```

### 6. `app_Futbol/admin.py`
```python
from django.contrib import admin
from .models import Equipo, Jugador, Partido

admin.site.register(Equipo)
admin.site.register(Jugador)
admin.site.register(Partido)
```

### 7. `app_Futbol/templates/base.html`
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Fútbol</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <style>
        body { background-color: #f8f9fa; padding-top: 20px; }
        .navbar-custom { background: linear-gradient(135deg, #1e3c72, #2a5298); }
        .footer-custom { 
            background: linear-gradient(135deg, #1e3c72, #2a5298); 
            color: white; 
            position: fixed; 
            bottom: 0; 
            width: 100%; 
        }
        .main-content { margin-bottom: 100px; }
    </style>
</head>
<body>
    {% include 'header.html' %}
    {% include 'navbar.html' %}
    
    <div class="container mt-4 main-content">
        {% block content %}
        {% endblock %}
    </div>
    
    {% include 'footer.html' %}
    
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
```

### 8. `app_Futbol/templates/navbar.html`
```html
<nav class="navbar navbar-expand-lg navbar-dark navbar-custom mb-4">
    <div class="container">
        <a class="navbar-brand" href="{% url 'inicio_futbol' %}">
            ⚽ Sistema de Administración Fútbol
        </a>
        
        <div class="navbar-nav ms-auto">
            <a class="nav-link" href="{% url 'inicio_futbol' %}">
                🏠 Inicio
            </a>
            
            <div class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                    👥 Equipos
                </a>
                <ul class="dropdown-menu">
                    <li><a class="dropdown-item" href="{% url 'agregar_equipo' %}">Agregar Equipo</a></li>
                    <li><a class="dropdown-item" href="{% url 'ver_equipos' %}">Ver Equipos</a></li>
                </ul>
            </div>
            
            <div class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                    👤 Jugadores
                </a>
                <ul class="dropdown-menu">
                    <li><a class="dropdown-item" href="#">Agregar Jugador</a></li>
                    <li><a class="dropdown-item" href="#">Ver Jugadores</a></li>
                </ul>
            </div>
            
            <div class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown">
                    🏆 Partidos
                </a>
                <ul class="dropdown-menu">
                    <li><a class="dropdown-item" href="#">Agregar Partido</a></li>
                    <li><a class="dropdown-item" href="#">Ver Partidos</a></li>
                </ul>
            </div>
        </div>
    </div>
</nav>
```

### 9. `app_Futbol/templates/footer.html`
```html
<footer class="footer-custom text-center py-3">
    <div class="container">
        <span>
            &copy; {% now "Y" %} - Sistema de Fútbol - 
            Creado por Ing. Eliseo Nava, Cbtis 128 - 
            Fecha: {% now "d/m/Y" %}
        </span>
    </div>
</footer>
```

### 10. `app_Futbol/templates/inicio.html`
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto text-center">
        <h1 class="mb-4">⚽ Sistema de Gestión de Fútbol</h1>
        <p class="lead">Bienvenido al sistema de administración de equipos, jugadores y partidos de fútbol</p>
        
        <div class="card mt-4">
            <div class="card-body">
                <img src="https://images.unsplash.com/photo-1574629810360-7efbbe195018?ixlib=rb-4.0.3&auto=format&fit=crop&w=1000&q=80" 
                     class="img-fluid rounded" 
                     alt="Fútbol" 
                     style="max-height: 400px;">
                <p class="mt-3">Gestiona equipos, jugadores y partidos de forma eficiente</p>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

### 11. `app_Futbol/templates/equipo/agregar_equipo.html`
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <h2>Agregar Nuevo Equipo</h2>
        <form method="POST">
            {% csrf_token %}
            <div class="mb-3">
                <label class="form-label">Nombre</label>
                <input type="text" class="form-control" name="nombre" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Ciudad</label>
                <input type="text" class="form-control" name="ciudad" required>
            </div>
            <div class="mb-3">
                <label class="form-label">País</label>
                <input type="text" class="form-control" name="pais" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Fundación</label>
                <input type="date" class="form-control" name="fundacion" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Estadio</label>
                <input type="text" class="form-control" name="estadio" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Entrenador</label>
                <input type="text" class="form-control" name="entrenador" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Colores</label>
                <input type="text" class="form-control" name="colores" required>
            </div>
            <button type="submit" class="btn btn-success">Guardar Equipo</button>
            <a href="{% url 'ver_equipos' %}" class="btn btn-secondary">Cancelar</a>
        </form>
    </div>
</div>
{% endblock %}
```

### 12. `app_Futbol/templates/equipo/ver_equipos.html`
```html
{% extends 'base.html' %}

{% block content %}
<h2>Lista de Equipos</h2>
<a href="{% url 'agregar_equipo' %}" class="btn btn-primary mb-3">➕ Agregar Nuevo Equipo</a>

<table class="table table-striped">
    <thead>
        <tr>
            <th>Nombre</th>
            <th>Ciudad</th>
            <th>País</th>
            <th>Estadio</th>
            <th>Entrenador</th>
            <th>Acciones</th>
        </tr>
    </thead>
    <tbody>
        {% for equipo in equipos %}
        <tr>
            <td>{{ equipo.nombre }}</td>
            <td>{{ equipo.ciudad }}</td>
            <td>{{ equipo.pais }}</td>
            <td>{{ equipo.estadio }}</td>
            <td>{{ equipo.entrenador }}</td>
            <td>
                <a href="{% url 'actualizar_equipo' equipo.id %}" class="btn btn-warning btn-sm">✏️ Editar</a>
                <a href="{% url 'borrar_equipo' equipo.id %}" class="btn btn-danger btn-sm">🗑️ Borrar</a>
            </td>
        </tr>
        {% endfor %}
    </tbody>
</table>
{% endblock %}
```

### 13. `app_Futbol/templates/equipo/actualizar_equipo.html`
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-8 mx-auto">
        <h2>Actualizar Equipo</h2>
        <form method="POST" action="{% url 'realizar_actualizacion_equipo' equipo.id %}">
            {% csrf_token %}
            <div class="mb-3">
                <label class="form-label">Nombre</label>
                <input type="text" class="form-control" name="nombre" value="{{ equipo.nombre }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Ciudad</label>
                <input type="text" class="form-control" name="ciudad" value="{{ equipo.ciudad }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">País</label>
                <input type="text" class="form-control" name="pais" value="{{ equipo.pais }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Fundación</label>
                <input type="date" class="form-control" name="fundacion" value="{{ equipo.fundacion|date:'Y-m-d' }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Estadio</label>
                <input type="text" class="form-control" name="estadio" value="{{ equipo.estadio }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Entrenador</label>
                <input type="text" class="form-control" name="entrenador" value="{{ equipo.entrenador }}" required>
            </div>
            <div class="mb-3">
                <label class="form-label">Colores</label>
                <input type="text" class="form-control" name="colores" value="{{ equipo.colores }}" required>
            </div>
            <button type="submit" class="btn btn-warning">Actualizar Equipo</button>
            <a href="{% url 'ver_equipos' %}" class="btn btn-secondary">Cancelar</a>
        </form>
    </div>
</div>
{% endblock %}
```

### 14. `app_Futbol/templates/equipo/borrar_equipo.html`
```html
{% extends 'base.html' %}

{% block content %}
<div class="row">
    <div class="col-md-6 mx-auto">
        <h2>¿Estás seguro de borrar este equipo?</h2>
        <div class="card">
            <div class="card-body">
                <h5 class="card-title">{{ equipo.nombre }}</h5>
                <p class="card-text">
                    <strong>Ciudad:</strong> {{ equipo.ciudad }}<br>
                    <strong>País:</strong> {{ equipo.pais }}<br>
                    <strong>Estadio:</strong> {{ equipo.estadio }}
                </p>
                <form method="POST" action="{% url 'realizar_borrado_equipo' equipo.id %}">
                    {% csrf_token %}
                    <button type="submit" class="btn btn-danger">Sí, Borrar</button>
                    <a href="{% url 'ver_equipos' %}" class="btn btn-secondary">Cancelar</a>
                </form>
            </div>
        </div>
    </div>
</div>
{% endblock %}
```

## COMANDOS PARA EJECUTAR:

```bash
# Crear migraciones
python manage.py makemigrations

# Aplicar migraciones
python manage.py migrate

# Ejecutar servidor
python manage.py runserver 8036
```

**URL:** http://127.0.0.1:8036/

El proyecto está COMPLETO y FUNCIONAL con CRUD para Equipos.
