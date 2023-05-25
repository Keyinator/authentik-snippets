# discord_oauth_get-user-info-and-avatar

## Description
- Puts discord's oauth info into the user's attributed ``discord``
- Assigns the discord-avatar as web-syntax in ``avatar`` for use with https://goauthentik.io/docs/installation/configuration#authentik_avatars

## Locations
| Flow | Stage |
| :--- | :--- |
| Enrollment | User Login Stage |
| Authentication | 	User Login Stage |

## Code
```py
import base64
import requests

avatar_url = 'https://cdn.discordapp.com/avatars/{id}/{avatar}.png?site={avatar_size}'
avatar_stream_content = 'data:image/png;base64,{base64_string}' #Converts base64 image into html syntax useable with authentik's avatar attributes feature

### CONFIG ###
avatar_size = "64" #Valid values: 16,32,64,128,256,512,1024
##############

def get_as_base64(url):
    """Returns the base64 content of the url
    """
    return base64.b64encode(requests.get(url).content)

def get_avatar_from_avatar_url(url):
    """Returns an authentik-avatar-attributes-compatible string from an image url
    """
    cut_url=f'{url}?size=64'
    return avatar_stream_content.format(base64_string=(get_as_base64(cut_url).decode("utf-8")))

user = request.user
userinfo = request.context['oauth_userinfo']

#Assigns the discord attributes to the user
user.attributes['discord'] = {
  'id': userinfo['id'],
  'username': userinfo['username'],
  'discriminator': userinfo['discriminator'],
  'email': userinfo['email'],
  'avatar': userinfo['avatar'],
  'avatar_url': avatar_url.format(id=userinfo['id'],avatar=userinfo['avatar'],avatar_size=avatar_size) if userinfo['avatar'] else None,
}

#If the user has an avatar, assign it to the user
user.attributes['avatar'] = get_avatar_from_avatar_url(user.attributes['discord']['avatar_url'])
 
user.save()
return True
```