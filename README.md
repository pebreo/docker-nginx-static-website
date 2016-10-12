Development
-----------
```

# make a development machine
dma create -d virtualbox dev1
deval dev1
dma ip dev1  # give the ip address

# stop all container
dallstop
dallrm

# rebuild and bring back up
dc build
dc up  # to show possible errors

cd web
pip install -r requirements.txt
./manage.py runserver_plus
```

Deployment
-----------
```
deval prod1
dc build
dc up -d

```


# Initial server setup


Step . Spin up both servers
----------------------------
```
docker-machine create \
-d digitalocean \
--digitalocean-access-token=DO_ACCESS_TOKEN \
--digitalocean-size=1gb \
prod1

docker-machine create \
-d digitalocean \
--digitalocean-access-token=DO_ACCESS_TOKEN \
--digitalocean-size=1gb \
dbprod

* Note ip address of db server
* Note ip address of web server
```

Step . Provision database server
-------------------------------
```
deval dbprod
cd db_machine
dc build
dc up -d
```


Step . Provision web server
----------------------------
```

Pre-provision checklist
* add database server ip to  .env file
* BRAINTREE_ENVIRONMENT = braintree.Environment.Production
* For social authentication, set your callback URL on Twitter/Facebook webapp configuration to be dashaccounting.com/complete/twitter or /complete/facebook
* DEBUG=False in your .env file
* make sure config.py is in django_social_app directory 
* make sure recaptcha site key for in templates/recaptcha/widget.html points to right domain

cd web
dc -f prod.yml build
dc -f prod.yml up -d 
dc -f prod.yml run --rm web sh create_superuser.sh
dc run --rm web /usr/local/bin/python manage.py collectstatic

Deployment/security checklist - Make sure that:
* Change admin password
* Uninstall werkzeug: dc run web pip uninstall -y werkzeug
* django settings: ALLOWED_HOSTS = ['*']
* The webhook on braintree is pointed to http://<domain>/aeotunhistEEhietqtbxEO/
* WORRY ABOUT LATER: in django settings: uncomment CACHE setting


```

Step . Harden db server using iptables and fail2ban
---------------------------------------------------
```
iptables -L --linenumbers

iptables -I DOCKER 1 -p tcp ! -s <ip_address> --dport 5432 -j DROP
apb -i hosts -e "box=<dbmachine> okhost=<db_ip>" harden.yml

# from web server
# zero is success, 1 is failure
nc -z -w5 <dbmach_ip> 5432; echo $?

# from your laptop
nc -z -w5 <dbmach_ip> 5432; echo $?
```

Step . Harden web server using fail2ban
--------------------------------------
```
apb -i hosts -e "box=<prodmachine> okhost=<prod1_ip>" harden.yml
```

Backup data
------------
```
deval <db_machine>
dma ssh <db_machine>
docker exec postgrescont pg_dump -U postgres -d postgres -f /tmp/backup.sql

TODO: clarify what these commands do
optional: from the host machine:
psql -h <postgrescont_ip> -p 5432 -U postgres --password

optional, another command:
docker run -it --name pgdumpcont -v /tmp/pgdumpcont:/tmp --volumes-from postgrescont postgres:latest bash
```


Configure logging
-----------------
```
dma ssh mybox

sudo su
vim /etc/rsyslog.d/10-docker.conf

# Docker logging
daemon.* {
 /var/log/docker.log
 stop
}

vim /etc/logrotate.d/docker

/var/log/docker.log {
    size 100M
    rotate 2
    missingok
    compress
}

service rsyslog restart


tail -f /var/log/docker.log
```


WEB
---


INSTALLATION
--------------
$ pip install -r requirements.txt
$ brew install redis

# start redis
$ redis-server

# test redis
$ redis-cli ping


# test celery
$ celery -A myproj beat -l info 


RUNNING TASKS
-------------
NOTE: In production, you will want to run thes command on supervisor.
See here for supervisor setup instructions:
https://realpython.com/blog/python/asynchronous-tasks-with-django-and-celery/

# run periodic tasks
$ celery -A myproj beat -l info 
or
$ celery -A myproj -B -l info


# run normal task
$ celery -A myproj worker -l info 


CELERY CRONTAB DOCS:
http://celery.readthedocs.org/en/latest/userguide/periodic-tasks.html#crontab-schedules



Installation
---------

Step 1. Install node package manager (npm) by going to `https://nodejs.org/` and click INSTALL

Step 2. Check that `npm` is installed:

```bash
npm -v
```

Step 3. Install gulp globally

```bash
npm install -g gulp

```

Step 4. Install requirements

```bash
cd myproject
npm install # this will create node_modules/ subdirectory in your directory
```

Run gulp + browsersync + django
------------------------------

Step 5. Run gulp
```bash
gulp # this will run django and open a browser
```

IMPORTANT: WHEN USING GULP+BROWSERSYNC, all your STDOUT+STDERR is in Chrome Console


Troubleshooting
------------
If you have trouble connecting, make sure the port is set to the correct port.
Trying closing your browser.
Also, you might have to goto the BrowserSync control panel (localhost:3001) and click 'Reload Browser' to refresh it.



# Model Notes

## Comments

What happens when you create a comment?

views.create_comment()
 -> notify.send()  # from apps.signals.notify

*The notify signal is defined in signals.py*
apps.notifications.models

*we new_notification function to notify signal*
def new_notification(..)
    ... create a Notification instance ...

notify.connect(new_notification)

model:
    class Comment(models.Models)
        user = ForeignKey(User)
        video = ForeignKey(Video)
        parent = ForeignKey("self")
        text = TextField
        active = BooleanField

## Notifications model

class Notification(models.Model):
    actor = ForeignKey(UserModel)

    verb = CharField
    
    action_object = GenericForeignKey(...)

    target = GenericForeignKey(...)

    recipient = ForeignKey(UserModel)

Model Queries:
    get_user(user)
    all_unread(user)
    all_read(user)
    get_recent_for_user(user)
    has_notifications(user)
Instance Queries:
    is_child(self)
    get_children(self)
    

How is the Notification model used to send notification?

This model record an actor, a verb,
a action_object and a target. Each of the above
must have an instance, id, and content_type to
be recorded properly.

We wanted to record the statement
<actor> did/verb with <action_object> to <target>

## Video

class Video(models.Model)
    title = CharField
    slug = SlugField
    order = PositiveInteger
    embed_code = CharField
    active = Boolean

    category = ForeignKey(Category, related_name='videos')
    # used for notification? or for demo purpose?
    tags = GenericRelation("TaggedItem")

## Category

class Category(models.Model)
    title
    slug
    description
    active


# for demo/education purpose only?
## TaggedItem
class TaggedItem(models.Model)
    tag = SlugField
    content_type = ForeignKey(ContentType)
    object_id = PositiveIntegerField
    content_object = GenericForeignKey()
# used by notifications

# Ajax to fetch JSON from backend

$('#notification_toggle').click(function(){
    $.ajax({
        type: "POST",
        url: "{% url 'ajax_view' %}",
        data: { csrfmiddlewaretoken: "{{csrftoken}}"},
        success: function(data) {
            $(data.notifications).each(function() {
                var link = this;
                $('#mydropdownclass').append(
                    "<li>" + link + "</li>"
                ')
            })
        },
        error: function(rs, e) {
            console.log(rs);
            console.log(e);
        }
    })
});

# Signal Notes

```
accounts/models.py
MyUser
 .post_save.connect() -> new_user_receiver
    Whenever a MyUser is saved we should:
    1. Create a UserProfile
    2. Create a notification
    3. Create a UserMechantId (along with connecting to braintree)

 .user_logged_in.connect() -> user_logged_in_receiver
     Whenever a MyUser user logs in we should:
     1. call update_membership_status() which 
        checks if a user has a subscription



analytics/models.py
  custom: page_view.connect() -> page_view_receiver
    Whenever a we have a view that calls page_view.send()
    we should record the page view in PageView table(s).



billing/models.py
 custom: membership_date_update.connect() -> membership_updates_receiver
      -> instance.update_status()
    Whenver Membership record is saved we should call update_status()
    which checks the date end of our member and changes is_member
    when necessary.
 
 custom: membership_dates_update.connect() -> membership_dates_update_receiver
    Whenever we membership_dates_update.send() is called
    we should add more days to the date_end in the Membership model


    
 videos/models.py
 Video
  .post_save.connect() -> video_post_save_receiver.py
   Whenever we save a video would should generate a slug.



notifications/models.py
  notify.connect() ->  new_notification
    Whenever notify.send() is called it should:
     1. Parse the arguments:
         a. recipient
         b. sender
         c. verb
         e. actor_content_type
         f. actor_id
         g. verb
         h. target
         i. action_object
     Save it to Notifications tables
```


# Braintree API notes

Setup
------
You should install braintree with `pip install braintree`
Then in your views.py you should add this to the top of the file:

```
import braintree
braintree.Configuration.configure(braintree.Environment.Sandbox,
                          merchant_id="93nj7zscx7jcppd6",
                          public_key="cbwmsm5gstck2mpk",
                          private_key="f8b54db855cae4751a9a230588cf5fb3")

SUBSCRIPTION_PLAN_ID = '4jm2'
```

Usage
----
First, create a customer
```
customer_result = braintree.Customer.create({})
customer_id = customer_result.customer.id
```

Next, create a payment method

```
nonce = request.POST.get("payment_method_nonce")
payment_result = braintree.PaymentMethod.create({
        "customer_id": customer_id,
        "payment_method_nonce": nonce,
        "options": {"make_default": True},
    })
```

Finally, create a subscription
```
payment_token = payment_result.payment_method.token
subscription_result = braintree.Subscription({
        "pay_method_token": payment_token,
        "plan_id": SUBSCRIPTION_PLAN_ID
    })
```

Here are some common properties you can get 
from the different results:
```
# payment types: 'paypal_account', 'credit_card'
payment_type = subscription_result.subscription.transactions[0].payment_instrument_type
transaction_id = subscription_result.subscription.transactions[0].id
subscription_id = subscription_result.subscription.id
subscription_amount = subscription_result.subscription.price
```

How does your braintree payment view work?
-----------------------------------------
* Start at upgrade() which passes context which contains the client token associated with a client_id
* On the frontend, we use the braintree javascript rendered form (which is bootstrap responsive)
* Also, the braintree form already comes with input validation built in
* When a user submits the form, they are directed to upgrade_view_formpost()
* It makes sure that users are user.is_authenticated()
* It calles get_usermerchant_customerid()
* It gets the nonce from the form
* It calls create_subscription(user, nonce) . The nonce is a symbol for the payment that person made
* That function then creates a braintree.PaymentMethod.create(customer_id, payment_method_nonce, options={'makedefault'=True})
* Then that function calls create_subscription_2 (which i have to rename to a better name)
* create_subscription_2(user, merchant_obj, payment_token):
 - checks: do they have a subscription and plan_id associated with this usermerchantobj? 
    - if yes:
       - is their subscription active?
           if yes:
              - raise SubscriptionExistsError
           if no:
              - we have to update their subscription to by doing: braintree.Subscription.update(subscription_id, {'payment_method_token':token})
              - and we sure make sure merchant_obj.user.is_member = True
    - if no, we create a new subscription

* If create_subscription()  is successful, then we call save_subscription_and_plan_info(). (btw, i should rename the first func to create_update_subscription())
* save_subscription_plan_info() records the subscription_id and plan_id to the UserMerchantId instance
* next, we record the transaction on our database and call create_transaction()
* finally, if saving the transaction is success we call update_membership_date() this 


# jQuery

*given a style*
<style>
    .another_class {
        display: none;
    }
</style>

*hiding elements
$('#myclass').click(function(e){
    e.preventDefault();
    $('#another_class').toggle();
})



Idea:

pytho-socialauth

source: https://realpython.com/blog/python/adding-social-authentication-to-django/

# settings.py
SOCIAL_AUTH_PIPELINE = (
    'social.pipeline.social_auth.social_details',
    'social.pipeline.social_auth.social_uid',
    'social.pipeline.social_auth.auth_allowed',
    'social.pipeline.social_auth.social_user',
    'social.pipeline.user.get_username',
    'social.pipeline.user.create_user',
    'apps.socialauth.pipeline.save_profile',  # <--- set the path to the function
    'social.pipeline.social_auth.associate_user',
    'social.pipeline.social_auth.load_extra_data',
    'social.pipeline.user.user_details'
)

# apps.socialauth.pipeline.save_profile.py
def save_profile(backend, user, response, *args, **kwargs):
    if backend.name == 'facebook':
        profile = user.get_profile()
        if profile is None:
            profile = Profile(user_id=user.id)
        profile.gender = response.get('gender')
        profile.link = response.get('link')
        profile.timezone = response.get('timezone')
        profile.save()

# login.html
{% if user and not user.is_anonymous %}
  <a>Hello, {{ user.get_full_name }}!</a>
  <br>
  <a href="/logout">Logout</a>
{% else %}
  <a href="{% url 'social:begin' 'twitter' %}?next={{ request.path }}">Login with Twitter</a>
{% endif %}

Idea:
TwitterUser
    following = ForeignKey("self", related_name='', null=True, blank=True)
    followed_by = ForeignKey("self", related_name='followers', null=True, blank=True)

{{block.super}}


Twitter social auth
keys: https://apps.twitter.com/app/9045268/keys
webapp link: https://apps.twitter.com/app/9045268
