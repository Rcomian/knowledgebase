[Home](../index.md)

## DACPAC

### User Handling

Problem - provide a service user that can be used to login to your data-tier application.

* User should exist and be created if not already there.
* User should have minimal permissions.
* User's password should not be stored in source control.
* User should remain in the database after an upgrade.
* Database should not be marked as "Changed" when upgrading.
* Multiple databases should be able to use the same user.

### Standard Approach

My preferred approach for database access is to use a stored procedures layer rather than table access. This is a fairly old school approach but doesn't affect the security model too much.

DACPAC's can have a `Security` folder which includes information for roles and users. The idea seems to be to create your users and roles in here like the following:

```SQL
-- Security/User.sql

CREATE LOGIN [MyLogin]
    WITH PASSWORD = N'MyPassword',
    DEFAULT_DATABASE = [MyDatabase],
    DEFAULT_LANGUAGE = [us_english],
    CHECK_EXPIRATION = OFF,
    CHECK_POLICY = OFF
GO

CREATE USER [MyLogin] FOR LOGIN [MyLogin] WITH DEFAULT_SCHEMA=[dbo];
GO

GRANT CONNECT TO [MyLogin];
GO

exec sp_addrolemember 'db_datareader', N'MyLogin'
GO

GRANT EXECUTE ON OBJECT::[dbo].[MyObject1] TO [MyLogin];
GO

GRANT EXECUTE ON OBJECT::[dbo].[MyObject2] TO [MyLogin];
GO

GRANT EXECUTE ON OBJECT::[dbo].[MyObject3] TO [MyLogin];
GO
```

[source](https://stackoverflow.com/questions/26163810/user-created-using-dacpac-cannot-log-on)

In this example we're also adding permissions to all the objects in this file.

I don't like this approach for the following reasons:

* The user's password is in source control.
* The user's langauge is defined universally.
* The object's are all listed in this file, which means adding an object is a change to 2 files. This leads to unnecessary merge conflicts.
* The user permissions are direct to the user, if we ever add a second user or hierarchy of users then this explodes. (but YAGNI)

Imagine, for example, that the user needs to have their password changed. We can do this on the server, but once we upgrade this DACPAC, the database gets marked as "Changed" and the password gets reset. We'd need to synchronise passwords with the DACPAC, perhaps even use the modified DACPAC to change the password in live.

The final issue with this is that it doesn't seem to work if you have a user shared between 2 DACPACs. This isn't a normal use case, but if you want to deploy two instances of the DACPAC on the server, for example, it would be. What happens is that the second deployment succeeds, but the user isn't added to the database. If you add them manually the database gets marked as "Changed" on upgrade.

This is also an issue if you try to ignore user management entirely. You then need to add your service user to the database after deployment. When upgrading, you get the "This database has changed" warning because of the user being added - then the user gets removed and nothing works until you remember to add the user back again.

So lets try to fix the issues we can.

### Use a role

The first thing I do is add a custom role to the database. This lets me handle permissions through the role first, then worry about user handling later.

```SQL
-- Security/Role.sql

CREATE ROLE service_role;
GO
```

We can create as many roles as we need here.

### Assign permissions locally

We can assign permissions to these in the object script itself, like the following:

```SQL
-- dbo/Stored Procedures/crud_RetrieveCars.sql

CREATE PROCEDURE crud_RetrieveCars
AS
BEGIN
  -- Never do unbounded queries like this
  SELECT * FROM Cars
END
GO

GRANT EXECUTE ON crud_RetrieveCars TO service_role
```

Here we can see the entire object definition and all its permissions in one place.

Don't forget to add the `GO` after procedure definition. Without that, the GRANT PERMISSION becomes part of the stored procedure itself and gets run every time it's called. Which doesn't work if you're using minimal permissions.

### Add the user to the role

I haven't yet found a way to do this nicely. However this way works. The one compromise is that the login must be added to the server security tab before the DACPAC is deployed.

What we do is make use of the post deployment scripts. This makes the upgrade come with a warning, but I think it's a better warning than "This database changed" which means investigation should be required.

The post deployment script looks like the following:

```SQL
-- PostDeployment/Security.sql

CREATE USER service_user FOR LOGIN service_user WITH DEFAULT_SCHEMA=[dbo];
GO

GRANT CONNECT TO service_user;
GO

ALTER ROLE service_role ADD MEMBER service_user;
GO
```

### Summary

Lets see how we did with our requirements:

* ~~User should exist and be created if not already there.~~ <span style="color:red">x</span>
* User should have minimal permissions. <span style="color:green">✓</span>
* User's password should not be stored in source control. <span style="color:green">✓</span>
* User should remain in the database after an upgrade. <span style="color:green">✓</span>
* Database should not be marked as "Changed" when upgrading. <span style="color:green">✓</span>
* Multiple databases should be able to use the same user. <span style="color:green">✓</span>

So it's not bad. Ideally I'd like to find a way to check for the server login and create a default one if it's not already there (with the change password flag set). But I can't see a way to do that cleanly just yet.
