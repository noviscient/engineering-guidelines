# Application

- [Logging exceptions](#logging-exceptions)
- [Distinguish between anticipated and unanticipated exceptions](#distinguish-between-anticipated-and-unanticipated-exceptions)
- [Minimise system clock calls](#minimise-system-clock-calls)
- [Modelling periods of time](#modelling-periods-of-time)

## Logging exceptions

Use `logger.exception` in except blocks but pass a useful message - don't just pass on the caught exception's message. Don't even format the exception's message into the logged message - Sentry will pick up the original exception automatically.

Doing this enables Sentry to group the logged errors together rather than treating each logged exception as a new error. See [Sentry's docs](https://docs.sentry.io/platforms/python/guides/logging/#usage) for further info.

Don't do this:

```python
try:
    do_something()
except Exception as e:
    logger.exception(str(e))
```

or this:

```python
try:
    do_something()
except Exception as e:
    logger.exception("Unable to do something: %s" % e)
```

Instead, do this:

```python
try:
    do_something()
except UnableToDoSomething:
    logger.exception("Unable to do something")
```

## Distinguish between anticipated and unanticipated exceptions

When calling functions that can raise exceptions, ensure your handling distinguishes between anticipated and unanticipated exceptions. It generally makes sense to use separate exception classes for anticipated exceptions and to log any other exceptions to Sentry:

For example:

```python
try:
    some_usecase.do_something()
except some_usecase.UnableToDoSomething:
    # We know about this failure condition. No code change is required so we
    # don't log the error to Sentry.
    pass
except Exception:
    # This is *unanticipated* so we log the exception to Sentry as some change is
    # required to handle this more gracefully.
    logger.exception("Unable to do something")
```

The rule of thumb is that anything logged to Sentry requires a code change to fix it. If nothing can be done (ie a vendor time-out), publish an application event instead.

## Minimise system clock calls

Avoid calls to the system clock in the domain layer of the application. That
is, calls to `localtime.now()`, `localtime.today()` etc. Think of such calls
like network or database calls.

Instead, prefer computing relevant datetimes or dates at the interface layer and
passing them in. This won't always be possible but often is.

This makes testing easier as you don't need to mock a system call. Your
functions will be purer with controlled inputs and outputs.

Avoid the pattern of using a default of `None` for a date/datetime parameter
then calling the system clock to populate it if no value is explicitly passed.
Instead of:

```py
def some_function(*, base_date=None):
    if base_date is None:
        base_date = datetime.date.today()
    ...
```

prefer the more explicit:

```py
def some_function(*, base_date):
```

which forces callers to compute the date they want to use for the function.
As suggested above, such system-clock calls should be reserved for the interface
layer of your application and the value passed though into the
business-logic/domain layers.

## Modelling periods of time

It's common for domain objects to model some period of time that defines when an
object is "active" or "valid". When faced with this challenge, prefer to use
_datetime_ fields where the upper bound is nullable and exclusive. Eg:

```python
class SomeModel(models.Model):
    ...
    active_from = models.DateTimeField()
    active_to = models.DateTimeField(null=True)
```

Specifically, try and avoid using `datetime.date` fields as these are more error-prone
due to implicit conversion of datetimes and complications from daylight-savings
time.

Further, whether using `date`s or `datetime`s, allowing the upper bound to be
exclusive allows zero-length periods to be modelled, which is often required
(even if it isn't obvious that will be the case at first).

Don't follow this rule dogmatically: there will be cases where the appropriate
domain concept is a date instead of a datetime, but in general, prefer to model
with datetimes.