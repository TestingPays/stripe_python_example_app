# Stripe Python Example Application

Integrated example application using [Stripe's Charges API](https://stripe.com/docs/api#create_charge).

## Requirements

Python 3.4.3 or later is required to run this application. You can have multiple Python versions (2.x and 3.x) installed on the same system without problems. See [Python Virtual Environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/) for more information.

## Running

See [requirements.txt](requirements.txt) above for required packages or install using pip install as shown here.

```bash
$ pip install -r requirements.txt
```

Prior to running the server it is necessary to add your Stripe keys, or use your Testing Pays keys.
Then run the server, using the manage.py file.

```bash
$ python manage.py runserver 8000 --settings=tp_python_stripe_example.settings.dev
```
## API Keys

```python
STRIPE_PUBLISHABLE = os.getenv('STRIPE_PUBLISHABLE', '<insert-your-publishable-stripe-key-here>')

# Testing Pays Configurations

# Use your Testing Pays API key here
STRIPE_SECRET = os.getenv('STRIPE_SECRET', '<insert-your-private-stripe-key-here>')
```

The [dev.py](tp_python_stripe_example/settings/dev.py) within the settings folder allows for testing settings to be kept separate from production settings. Update both the `STRIPE_PUBLISHABLE` key that you got from Stripe and `STRIPE_SECRET` key available from Testing Pays within the `dev.py` file. The Stripe base url is also updated within the dev settings file to point to the Testing Pays API.

```javascript
Stripe.setPublishableKey('<insert-your-publishable-stripe-key-here>');
```

In addition, update the `Stripe.setPublishableKey` with your Stripe Publishable key in the [charges.js](python_stripe_payment/static/js/charges.js)
file.

## Using it with Testing Pays

[Testing Pays](http://www.testingpays.com) lets you test and simulate more than you would be able to do with regular API sandboxes. The application is setup so you can avail using your [Testing Pays API key](https://admin.testingpays.com) to test and prepare for errors, validation issues, server errors or even network outages.

```python
# Testing Pays Configurations
TESTINGPAYS_API_KEY = "<insert-your-testing-pays-api-key-here>"

# ...

STRIPE_SECRET = os.environ.get("STRIPE_SECRET", TESTINGPAYS_API_KEY)
# Set the base URL to Testing Pays API
STRIPE_BASE_URL = "https://api.testingpays.com/{0}/stripe/v1/charges".format(TESTINGPAYS_API_KEY)
```

## Production versus Development Environments

```python
@require_http_methods(['POST'])
def charges(request):
    if settings.STRIPE_BASE_URL:
        stripe.api_base = settings.STRIPE_BASE_URL

    try:
        stripe.Charge.create(
            amount=remove_decimal_places(extract_amount(request)),
            currency='EUR',
            description='Charge for testing.pays@example.com',
            source=request.POST['stripeToken'],
        )
    except stripe.error.CardError as error:
```

As shown in the code snippet above (see [views.py](python_stripe_payment/views.py)), the `charges` route checks if the `stripe.api_key` has been set prior to creating the Stripe charge. This `api_key` is only set for development evironments. When running in production the `STRIPE_BASE_URL` will be empty and therefore the default Stripe key will be used.
