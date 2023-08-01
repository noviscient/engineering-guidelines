# Django

- [Django project files structure](#django-project-struct)
- [`ChartField` choices](#chartfield-choices)
- [Model field naming conventions](#model-field-naming-conventions)
- [Model method naming conventions](#model-method-naming-conventions)
- [Model class naming conventions](#model-class-naming-conventions)
- [Model's meta class](#model-meta-class)
- [Encapsulate model mutation](#encapsulate-model-mutation)
- [Ensure `__str__` is unique](#unique-str)
- [Use database constraints for enforcing data integrity](#integrity-at-the-database)
- [Avoid field validators](#field-validators)
- [Avoid multiple domain calls from an interface component](#one-domain-call)
- [Use database constraints for enforcing data integrity](#integrity-at-the-database)


## <a name="django-project_struct">Django project files structure</a>

Prefered Django project files structure:

```
my_project/
├── my_app/
│   ├── migrations/
│   ├── static/
│   │   ├── css/
│   │   ├── js/
│   │   └── images/
│   ├── templates/
│   │   └── my_app/
│   │       └── (HTML templates)
│   ├── utils/
│   │   ├── billing.py
│   │   ├── import_resources.py
│   │   └── (other utility modules)
│   ├── tests/
│   │   ├── test_views.py
│   │   ├── test_models.py
│   │   └── (other test files)
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   └── urls.py
├── manage.py
├── my_project/
│   ├── settings.py
│   ├── urls.py
│   └── other_project_files...
├── static/
│   ├── (project-level static files)
├── templates/
│   ├── base.html
│   └── (other shared templates)
└── media/
    ├── (user uploaded files)

```

## <a name="chartfield-choices">`ChartField` choices</a>

The values stored in the database should be:

- Uppercase and separated with underscores.
- Namespaced with a string prefix.

A human-readable version should also be added in the tuples provided to the field.

```python
class ChannelChoices(models.TextChoices):
    TELESALES = "TELESALES", "Telesales"
    FIELD_SALES = "FIELD_SALES", "Field-sales"

...


class MyModel(models.Model):
    channel = models.CharField(max_length=128, choices=ChannelChoices.choices)
```

This is because the database value is a code or symbol intended to be used
within application logic but not shown to the end user - making it uppercase
makes this distinction clear. Using a human-readable version for the database
value can lead to bugs when a future maintainer wants to change the version
shown to the end user.

## <a name="model-field-naming-conventions">Model field naming conventions</a>

`DateTimeField`s should generally have suffix `_at`. For example:

- `created_at`
- `sent_at`
- `period_starts_at`

There are some exceptions such as `available_from` and `available_to` but stick
with the convention unless you have a very good reason not to.

`DateField`s should have suffix `_date`:

- `billing_date`
- `supply_date`

This convention also applies to variable names.

## <a name="model-method-naming-conventions">Model method naming conventions</a>

- For query methods (i.e. methods that look something up and return it), prefix with `get_`.

- For setter methods (i.e. methods that set fields and call save), prefix with `set_`.

- Prefer "latest" to "last" in method names as "latest" implies chronological order where the
  ordering is not explicit when using "last"; i.e.
  `get_latest_payment_schedule`. Similarly, prefer `earliest` to `first` in
  method names.

## <a name="model-class-naming-conventions">Model class naming conventions</a>

- Model class names should be in the singular form. For example, use "Subscription" instead of "Subscriptions."
- Model class names should be in CamelCase. For example, use "Subscription" instead of "subscription."
- Avoid using overly generic names like "Object" or "Item." Be more specific to avoid confusion and clashes with other model names in the future.

## <a name="model-meta-class">Model's meta class</a>

The `class Meta` should appear after the fields are defined, with a single blank line separating the fields and the class definition.

```py
class Person(models.Model):
    first_name = models.CharField(max_length=20)
    last_name = models.CharField(max_length=40)

    class Meta:
        verbose_name_plural = 'people'
```

The order of model inner classes and standard methods should be as follows (noting that these are not all required):

- All database fields
- Custom manager attributes
- class Meta
- def __str__()
- def save()
- def get_absolute_url()
- Any custom methods

## Encapsulate model mutation

Don't call a model's `save` method from anywhere but "mutator" methods on the
model itself.

Similarly, avoid calling `SomeModel.objects.create` or even
`SomeModel.related_objects.create` from outside of the model itself. Encapsulate
these in "factory" methods (classmethods for the `objects.create` call).

Doing this provides a useful overview of the lifecycle of a model as you
can see all the ways it can mutate in one place.

Also, this practice leads to better tests as you have a simple, readable method
to stub when testing units that call into the model layer.

Further reading:

- [Django models, encapsulation and data integrity](https://www.dabapps.com/blog/django-models-and-encapsulation/), by Tom Christie

## <a name="unique-str">Ensure `__str__` is unique</a>

Ensure the string returned by a model's `__str__` method uniquely identifies
that instance.

This is important as Sentry (and other tools) often just print `repr(instance)`
of the instance (which prints the output from `__str__`). When debugging, it's
important to know exactly which instances are involved in an error, hence why
this string should uniquely identify a single model instance.

It's fine to use something like:

```py
def __str__(self):
    return f"#{self.id} ..."
```

## <a name="field-validators">Avoid field validators</a>

[Django's field validators](https://docs.djangoproject.com/en/4.0/ref/validators/) should not be used.
Like model forms, they can be useful in an early stage project, but are better avoided.
This is enforced by flake8 rule `K306`.

Why? They make it look like a field will always have its value validated when in practice it will not.

Example:

```python
from django.core import validators
from django.db import models


class SomeModel(models.Model):
    constrained_number = models.FloatField(
        default=0.5,
        validators=[
            validators.MinValueValidator(0.0),
            validators.MaxValueValidator(1.0),
        ],
    )
```

This looks like a field that cannot be set to values below 0 or above 1.
But the validation exists in code rather than the database, so it is possible to store values outside this range.

Django only runs field validators when calling `.full_clean()` on a Model.
This is done when using Django Admin to build forms or when using model forms.
Neither of these are the recommended method of creating forms in Kraken.
They are also not pulled into the form fields when we use `forms.fields_for_model`.
As such, field validators are not run automatically, while looking like they are.

Validation should instead be performed explicitly when validating input.
[Constraints](#integrity-at-the-database) should be used to provide data integrity in the database.
This makes it clear when validation is and isn't being run.


## <a name="integrity-at-the-database">Use database constraints for enforcing data integrity</a>

Don't rely on the Python application for data integrity. Instead, use database
constraints to prevent the storage of corrupt data.

Application-level validations aimed at the user experience (such as form
validation) are acceptable as long as they aren't used as substitutes for data
integrity, but as an extra.

Some Django `QuerySet` methods such as `bulk_update()`, `bulk_create()`, and
`update()` don't call the model's `save()` method. If a model has validation
logic living in `save()`, it becomes susceptible to data inconsistencies.

The same risks apply to functions [encapsulating model mutations](#encapsulate-model-mutation)
and to other application-level validators, such as [model field validators](#field-validators).

When asserting data integrity is important, don't do this:

```python
def save(self, *args, **kwargs) -> None:
    if self.field_a is None and self.field_b is None:
        raise exceptions.ValidationError(
            "At least one of field_a or field_b must have a value."
        )
    return super().save(*args, **kwargs)
```

Assert it at the database level instead:

```python
class MyModel(models.Model):
    # model fields...

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=~Q(field_a__isnull=True, field_b__isnull=True),
                name="at_least_one_of_field_a_or_field_b_not_null",
            )
        ]
```

## <a name="one-domain-call">Avoid multiple domain calls from an interface component</a>

An interface component, like a view class/function or Celery task, should only make one
call into the domain layer (i.e. the packages in your codebase where application
logic lives).

Recall: the job of any interface component is this:

1. Translate requests from the language of the interface (i.e. HTTP requests or serialized Celery task payloads)
   into domain objects (e.g. `Account` instances)

2. Call into the domain layer of your application to fetch some query results or
   perform an action.

3. Translate any returned values (or exceptions) from the domain layer back into the
   language of the interface (e.g. into a 200 HTTP response for a successful query,
   or a 503 HTTP response if an action wasn't possible). Note, this step only
   applies to interfaces components that can _respond_ in some way - this
   doesn't apply to fire-and-forget Celery tasks where there's no results
   back-end.

This convention mandates that step 2 involves a single call into the domain
layer to keep the interface easy-to-maintain and to avoid leaking application
logic into the interface layer.

So avoid this kind of thing:

```python
class SomeView(generic.FormView):
    def form_valid(self, form):
        account = form.cleaned_data["account"]
        payment = form.cleaned_data["payment"]

        result = some_domain_module.do_something(account, payment)
        if result:
            some_other_domain_module.do_something_else(account)
        else:
            form.add_error(None, "Couldn't do something")
            return self.form_invalid(form)

        some_logging_module.log_event(account, payment)

        return shortcuts.redirect("success")
```

where the view class is making multiple calls into the domain layer.

Instead, encapsulate the domain functionality that the interface requires in a
single domain call:

```python
class SomeView(generic.FormView):
    def form_valid(self, form):
        try:
            some_domain_module.do_something(
                account=form.cleaned_data["account"],
                payment=form.cleaned_data["payment"],
            )
        except some_domain_module.UnableToDoSomething as e:
            # Here we assume the exception message is suitable for end-users.
            form.add_error(None, str(e))
            return self.form_invalid(form)

        return shortcuts.redirect("success")
```

Note the use of domain-specific exceptions to handle failure conditions.

## <a name="drf-serializers">DRF serializers</a>

Serializers provided by Django-REST-Framework are useful, not just for writing
REST APIs. They can be used anywhere you want to clean and validate a nested
dictionary of data. Here are some conventions for writing effective serializers.

Be liberal in what is accepted in valid input.

- Allow optional fields to be omitted from the payload, or have `null` or `""`
  passed as their value.

- Convert inputs into normal forms (by e.g. stripping whitespace or upper-casing).

In contrast, be conservative in what is returned in the `validated_data` dict.
Ensure `validated_data` has a consistent data structure no matter what valid
input is used. Don't make calling code worry about whether a particular key _exists_ in the
dict.

In practice, this means:

- Optional fields where `None` is not a valid value have a default value set in
  the serializer field declaration.

- If optional string-based fields have `allow_null=True`, then convert any
  `None` values to the field default.

To that end, use this snippet at the end of your `validate` method to ensure
there are no missing keys in the `validated_data` dict and that `None` is only
included as a value in `validated_data` when it's a field's default.

```py
def validate(self, data):
    ...
    # Ensure validated data includes all fields and None is only used as a value when it's
    # the default value for a field.
    for field_name, field in self.fields.items():
        if field_name not in data:
            data[field_name] = field.initial
        elif data[field_name] is None and data[field_name] != field.initial:
            data[field_name] = field.initial

    return data

```

Misc:

- Ensure validation error messages end with a period.

## <a name="integrity-at-the-database">Use database constraints for enforcing data integrity</a>

Don't rely on the Python application for data integrity. Instead, use database
constraints to prevent the storage of corrupt data.

Application-level validations aimed at the user experience (such as form
validation) are acceptable as long as they aren't used as substitutes for data
integrity, but as an extra.

Some Django `QuerySet` methods such as `bulk_update()`, `bulk_create()`, and
`update()` don't call the model's `save()` method. If a model has validation
logic living in `save()`, it becomes susceptible to data inconsistencies.

The same risks apply to functions [encapsulating model mutations](#encapsulate-model-mutation)
and to other application-level validators, such as [model field validators](#field-validators).

When asserting data integrity is important, don't do this:

```python
def save(self, *args, **kwargs) -> None:
    if self.field_a is None and self.field_b is None:
        raise exceptions.ValidationError(
            "At least one of field_a or field_b must have a value."
        )
    return super().save(*args, **kwargs)
```

Assert it at the database level instead:

```python
class MyModel(models.Model):
    # model fields...

    class Meta:
        constraints = [
            models.CheckConstraint(
                check=~Q(field_a__isnull=True, field_b__isnull=True),
                name="at_least_one_of_field_a_or_field_b_not_null",
            )
        ]
```