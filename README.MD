# Issue description
Spyne receives an empty body when running on gUnicorn in combination with DataDog tracing. This causes all our SOAP endpoints to break, since they now all cause an internal server error before even reaching our own code.

## Stack trace
```python
Internal Server Error: /soap/
Traceback (most recent call last):
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/django/core/handlers/exception.py", line 55, in inner
    response = get_response(request)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/django/core/handlers/base.py", line 197, in _get_response
    response = wrapped_callback(request, *callback_args, **callback_kwargs)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/ddtrace/contrib/trace_utils.py", line 334, in wrapper
    return func(mod, pin, wrapped, instance, args, kwargs)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/ddtrace/contrib/django/patch.py", line 302, in wrapped
    return func(*args, **kwargs)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/django/views/decorators/csrf.py", line 56, in wrapper_view
    return view_func(*args, **kwargs)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/server/django.py", line 88, in __call__
    response = WsgiApplication.__call__(self, environ, start_response)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/server/wsgi.py", line 310, in __call__
    return self.handle_rpc(req_env, start_response)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/server/wsgi.py", line 419, in handle_rpc
    contexts = self.generate_contexts(initial_ctx, in_string_charset)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/server/_base.py", line 69, in generate_contexts
    self.app.in_protocol.create_in_document(ctx, in_string_charset)
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/protocol/soap/soap11.py", line 203, in create_in_document
    ctx.in_document = _parse_xml_string(ctx.in_string,
  File "/Users/jur/Library/Caches/pypoetry/virtualenvs/ddtrace-django-spyne-reproduction-B6BBqRKv-py3.10/lib/python3.10/site-packages/spyne/protocol/soap/soap11.py", line 96, in _parse_xml_string
    chunk = next(xml_string)
StopIteration
```

## Steps to reproduce
1. Install poetry and python 3.10.13
2. Run `poetry install --no-root`
3. Run the project with `poetry run ddtrace-run gunicorn python manage.py runserver`
4. Do a SOAP request against `http://127.0.0.1:8000/soap`
