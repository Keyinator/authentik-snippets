# discord_oauth_assign-roles-of-guild-as-authentik-roles_authentication

## Description
Assigns roles of a specific guild as authentik groups based on a map: Discord_Role_ID<>Authentik_Role_Name

## Location
| Flow | Stage |
| :--- | :--- |
| Enrollment | User Write Stage |

## Warnings
- Discord Social login needs the following permission: ``guilds.members.read``

## Code
```py
from authentik.core.models import Group
GUILD_API_URL = "https://discord.com/api/users/@me/guilds/{guild_id}/member"

### CONFIG ###
guild_id = "<YOUR GUILD ID>"
mapped_roles = {
  "<Discord Role Id>": Group.objects.get_or_create(name="<Authentik Role Name>")[0],
}
##############

# Ensure flow is only run during oatuh logins via Discord
if context['source'].provider_type != "discord":
  ak_message(f"Wrond provider: {context['source'].provider_type}")
  return False

# Get the user-source connection object from the context, and get the access token
connection = context['goauthentik.io/sources/connection']
access_token = connection.access_token

guild_member_info = requests.get(
  GUILD_API_URL.format(guild_id=guild_id),
  headers={
    "Authorization": "Bearer " + access_token
  },
).json()

# Ensure user is a member of the guild
if "code" in guild_member_info:
    if guild_member_info['code'] == 10004:
        ak_message("User is not a member of the guild")
    else:
        ak_create_event("discord_error", source=context['source'], code=guild_member_info['code'])
        ak_message("Discord API error, try again later.")
    return False

# Add all mapped roles the user has in the guild
groups_to_add = []
for role_id in mapped_roles:
  if role_id in guild_member_info['roles']:
    groups_to_add.append(mapped_roles[role_id])

request.context["flow_plan"].context["groups"] = groups_to_add
return True
```