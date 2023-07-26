# Aliases
- [API documentation](#api-documentation)
- [Query documentation](#query-documentation)
- [Python Regular Expressions](Python-Regular-Expressions) Match function
- [Definitions](#definitions)
- [Create an alias for a target](#create-an-alias-for-a-target)
- [Update an alias to point to a new target](#update-an-alias-to-point-to-a-new-target)
- [Delete an alias regardless of the target](#delete-an-alias-regardless-of-the-target)
- [Remove aliases from a specified target](#remove-aliases-from-a-specified-target)
- [Delete all of a user's aliases](#delete-all-of-a-users-aliases)
- [Display aliases](#display-aliases)
- [Bulk delete aliases](#bulk-delete-aliases)
- [Bulk reassign aliases](#bulk-reassign-aliases)
- [Determine if an address is a user, user alias, group or group alias](#determine-if-an-address-is-a-user-user-alias-group-or-group-alias)

## API documentation
* https://developers.google.com/admin-sdk/directory/reference/rest/v1/users.aliases
* https://developers.google.com/admin-sdk/directory/reference/rest/v1/groups.aliases

## Query documentation
* https://developers.google.com/admin-sdk/directory/v1/guides/search-users

## Definitions
```
<DomainName> ::= <String>(.<String>)+
<EmailAddress> ::= <String>@<DomainName>
<EmailAddressList> ::= "<EmailAddress>(,<EmailAddress>)*"
<EmailAddressEntity> ::= <EmailAddressList> | <FileSelector> | <CSVkmdSelector> | <CSVDataSelector>
<UniqueID> ::= id:<String>
```
## Create an alias for a target
```
gam create alias|aliases <EmailAddressEntity> user|group|target <UniqueID>|<EmailAddress>
        [verifynotinvitable]
```
`<EmailAddressEntity>` are the aliases, `<EmailAddress>` is the target.

The `verifynotinvitable` option causes GAM to verify that the alias email address being created is not that of an unmanaged account;
if it is, the command is not performed.

### Example

To allow Robert to also receive mail as Bob:

```
gam create alias bob[@yourdomain.com] user robert[@yourdomain.com]
```

## Update an alias to point to a new target
The existing alias is deleted and a new alias is created.
```
gam update alias|aliases <EmailAddressEntity> user|group|target <UniqueID>|<EmailAddress>
        [notargetverify]
```
`<EmailAddressEntity>` are the aliases, `<EmailAddress>` is the target.

By default, GAM makes additional API calls to verify that the target email address exists before updating the alias;
if you know that the target exists, you can suppress the verification with `notargetverify.

## Delete an alias regardless of the target
```
gam delete alias|aliases [user|group|target] <EmailAddressEntity>
```
`<EmailAddressEntity>` are the aliases.

## Remove aliases from a specified target
```
gam remove alias|aliases <EmailAddress> user|group <EmailAddressEntity>
```
`<EmailAddress>` is the target, `<EmailAddressEntity>` are the aliases.

## Delete all of a user's aliases
```
gam <UserTypeEntity> delete aliases
```

## Display aliases
Display a specific alise.
```
gam info alias|aliases <EmailAddressEntity>
```

Display selected aliases.
```
gam print aliases [todrive <ToDriveAttribute>*]
        [domain <DomainName>] [(query <QueryUser>)|(queries <QueryUserList>)]
        [user|users <EmailAddressList>] [group|groups <EmailAddressList>]
        [select <UserTypeEntity>]
        [aliasmatchpattern <RegularExpression>]
        [shownoneditable] [nogroups] [nousers]
        [onerowpertarget] [delimiter <Character>]
        [suppressnoaliasrows]
        (addcsvdata <FieldName> <String>)*
```
By default, group and user aliases in all domains in the account are selected; these options allow selection of subsets of aliases:
* `domain <DomainName>` - Limit aliases to those in `<DomainName>`
* `(query <QueryUser>)|(queries <QueryUserList>)` - Print aliases for selected users
* `user|users <EmailAddressList>` - Print aliases for users in `<EmailAddressList`
* `select <UserTypeEntity>` - Print aliases for users in `<UserTypeEntity>`
* `group|groups <EmailAddressList>` - Print aliases for groups in `<EmailAddressList`
* `aliasmatchpattern <RegularExpression>` - Print aliases that match a pattern
* `nogroups` - Print only user aliases
* `nousers` - Print only group aliases

By default, the CSV output has three columns: `Alias,Target,TargetType`; if a target
has multiple aliases, there will be multiple rows, one per alias.

Use `shownoneditable` to list non-editable alias email addresses; these are typically outside of the account's primary domain or subdomains.
This adds the column `NonEditableAlias`.

Specifying `onerowpertarget` changes the three columns to: `Target,TargetType,Aliases`; all aliases for the target are listed in the
`Aliases` column. If `shownoneditable` is specified, there will be a fourth column `NonEditableAliases` with a list of non-editable aliases.

By default, the aliases in a list are separated by the `csv_output_field_delimiter' from `gam.cfg`.
* `delimiter <Character>` - Separate aliases in a list with `<Character>`

Specifying both `onerowpertarget` and `suppressnoaliasrows` causes GAM to not display any targets that have no aliases.

## Bulk delete aliases
You can bulk delete aliases as follows; use `(query <QueryUser>)|(queries <QueryUserList>)` and
`aliasmatchpattern <RegularExpression>` as desired.
```
gam redirect csv ./OldDomainAliases.csv print aliases aliasmatchpattern ".*@olddomain.com" onerowpertarget suppressnoaliasrows
gam redirect stdout ./DeleteAliases.txt multiprocess redirect stderr stdout csv ./OldDomainAliases.csv gam remove aliases "~Target" "~TargetType" "~Aliases"
```

## Bulk reassign aliases
You can bulk reassign aliases as follows. Make a CSV file ReassignAliases.csv with two columns: OldTarget,NewTarget.
From this CSV file, all of the aliases for the users in the OldTarget column will be listed with an additional column showing the NewTarget.
```
gam redirect stdout ./GetAliases.txt multiprocess redirect stderr stdout redirect csv ./ReassignAliases.csv gam print aliases user "~OldTarget" addcsvdata NewTarget "~NewTarget"
```
If an OldTarget's aliases are to be reassigned to more than the one NewTarget, edit ReassignAliases.csv and make changes as required.
```
gam redirect stdout ./ReassignAliases.txt multiprocess redirect stderr stdout csv ReassignAliases.csv gam update alias "~Alias" user "~NewTarget"
```

## Determine if an address is a user, user alias, group or group alias
```
gam whatis <EmailItem> [noinfo] [noinvitablecheck]
```
The first line of output is: `<TypeOfEmailItem>: <EmailItem>`

There is additional output based on `<TypeOfEmailItem>`:
* User - `gam info user <EmailItem>`
* Group - `gam info group <EmailItem>`
* User Alias - `gam info alias <EmailItem>`
* Group Alias - `gam info alias <EmailItem>`
* User Invitation - `gam info userinvitation <EmailItem>`

The `noinfo` argument suppresses the additional output.

The `noinvitablecheck` argument suppresses the user invitation check
to avoid exceeding quota limits when checking a large number of addresses.

The return code is set based on `<TypeOfEmailItem>`:
* User - 20
* User Alias - 21
* Group - 22
* Group Alias - 23
* User Invitation - 24
* Unknown - 59
