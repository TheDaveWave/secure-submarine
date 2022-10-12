# Secure Submarine
This version uses React, Redux, Express, Passport, and PostgreSQL (a full list of dependencies can be found in `package.json`). This version uses React to control the login requests and redirection in coordination with client-side routing.

## Use the Download ZIP Option (Don't Fork & Don't Clone)

**Step 1 - Download ZIP of Project:**
- click on the **Code** button
    - select the **Download ZIP** option from the menu
- file download should appear at the bottom of your web browser
    - click on the downloaded file icon
- a finder window should open with the assignment folder unzipped in the Downloads directory
- move the folder to where you are keeping your projects

**Step 2 - Initialize as a Repo:**
- open the project in VSCode
- open up the terminal in VSCode for the project
    - *in terminal:* run `git init`
    - *in terminal:* run `git add .`
    - *in terminal:* run `git commit -m "initial commit"`

**Step 3 - Attach GitHub Repo:**
- goto GitHub in the browser (make sure you are logged in)
- click on the `+` icon in the top right of the window
    - select **New repository** from the menu
- name your repo **secure-submarine**
- make sure your repo has **Public** selected
- click the **Create repository** button at the bottom
- copy the terminal commands listed in the **…or push an existing repository from the command line** section
- goto your project terminal in VSCode and paste the commands you just copied from GitHub and press **return**


---

## Assignment Instructions

### Base Mode

On the Secure Submarine, there are many secrets, but our enemies are out to steal our secrets! We just realized that our Secure Submarine web portal (the portal for all of the hottest gossip on the secure submarine) is compromised! Anyone, logged in or not, can visit `http://localhost:5000/api/secrets` to see all of the secrets for the entire crew!

#### No Secrets for the Unauthenticated (Authentication)

> Task: Only users who are authenticated should see any secrets.

The user router is protected from unauthenticated requests thanks to the `rejectUnauthenticated` middleware:

```JavaScript
router.get('/', rejectUnauthenticated, (req, res) => {
  res.send(req.user);
});
```

Do this for the secrets route to achieve these results:

- [ ] An unauthenticated user visiting `http://localhost:5000/api/secrets` should get a `403` or `forbidden` error instead of seeing the secrets.
- [ ] A user like `Admiral Greer` with password `tuna` should still be able to visit `http://localhost:3000/#/secrets` to see all of the secrets.

#### No Secrets Above Clearance Level (Authorization)

> Task: When authenticated, a user should only see secrets with a `secrecy_level` that is equal or less than the user's `clearance_level`.

A user like `Captain Borodin` with password `shark` is be to visit `http://localhost:3000/#/secrets` to see all of the secrets! That's no good! There's a secret in there that calls him weird!

You should see the clearance level in the console log inside of secrets GET request in `secrets.router.js`. Now fix the query in `secrets.router.js` so that it uses the clearance level to determine which secrets to return.

- [ ] A user like `Captain Borodin` with password `shark` should no longer be able to see any secrets above his `clearance_level` which is `10`.
- [ ] A user like `Admiral Greer` with password `tuna` should still be able to visit `http://localhost:3000/#/secrets` to see all of the secrets.

### Stretch Goals

#### Hashing

If you're interested in securely storing passwords, you should salt and hash them. Here is a quick video that explains it pretty well: https://www.youtube.com/watch?v=8ZtInClXe1Q

Right now, we are storing the passwords in plain text, so if the enemy got a hold of our database, they would know everybody's password! Instead of storing plain passwords, we should scramble them up. That is called hashing.

Uncomment this line to start hashing passwords for each user.

```JavaScript
return bcrypt.hashSync(password, '$2b$10$p5Wkte33hlOBOcUtJie6H.');
```

Because the database doesn't store the actual password, we can't just check to see if they are equal like we were doing before. Uncomment this line to allow us to check `candidatePasswords` (what the user entered), against the `storedPassword` (the hash in the database).

```JavaScript
return bcrypt.compareSync(candidatePassword, storedPassword);
```

New users will now have their passwords hashed!

Run these queries to add your users back to the database with hashed passwords:

```SQL
DROP TABLE "user";

CREATE TABLE "user" (
    "id" SERIAL PRIMARY KEY,
    "username" VARCHAR (80) UNIQUE NOT NULL,
    "password" VARCHAR (1000) NOT NULL,
    "clearance_level" INTEGER NOT NULL DEFAULT 0
);

INSERT INTO "user" ("username", "password", "clearance_level")
VALUES ('Admiral Greer', '$2b$10$p5Wkte33hlOBOcUtJie6H.PnCvk8v.KjZspVoAFtT7g5v5xK.EXVG', 18),
('Captain Borodin', '$2b$10$p5Wkte33hlOBOcUtJie6H.ZIgFjzr4zY8FItxC8gZyqIWD5gYmL0m', 10),
('Lieutenant Nguyen', '$2b$10$p5Wkte33hlOBOcUtJie6H.vaUd5ikB1LWCbVZAA87BR63NiDorn1C', 4),
('Lieutenant Ryan', '$2b$10$p5Wkte33hlOBOcUtJie6H.PnCvk8v.KjZspVoAFtT7g5v5xK.EXVG', 4);
```

#### Salting
Now that we are no longer storing plain text passwords. The enemy is unable to see the crew's passwords. However, Lieutenant Ryan has been careless, and the enemy knows that his password is `tuna`. Because of this, they can see the lowest security information. Then they notice that Admiral Greer's hashed password perfectly matches Lieutenant Ryan's hashed password! They now know that Admiral Greer's password is `tuna` as well! We should fix our code so that even if two people have the same password, it has a different hash in the database. Enter salting! Salting is the process of generating a random string for each user. Notice that every password starts with `$2b$10$p5Wkte33hlOBOcUtJie6H.`. That is the salt! These should be random and generated uniquely for each user.

Uncomment these two lines to start creating a unique salt for each user.

```JavaScript
const salt = bcrypt.genSaltSync(SALT_WORK_FACTOR); // This generates a random salt
```

```JavaScript
return bcrypt.hashSync(password, salt);
```

New users will now have their passwords salted and hashed!

Run these queries to add your users back to the database with hashed passwords:

```SQL
DROP TABLE "user";

CREATE TABLE "user" (
    "id" SERIAL PRIMARY KEY,
    "username" VARCHAR (80) UNIQUE NOT NULL,
    "password" VARCHAR (1000) NOT NULL,
    "clearance_level" INTEGER NOT NULL DEFAULT 0
);

INSERT INTO "user" ("username", "password", "clearance_level")
VALUES ('Admiral Greer', '$2b$10$uxPm0qeJAz70oqhEg8dX6uXlYc2PWUtPuZhTa65OiDv2LCHA41OLq', 18),
('Captain Borodin', '$2b$10$iUCrWSMvLpYuKQLsmmTiNe3gfU6jAdyElCbCLtboVH6DlXJdsuPxG', 10),
('Lieutenant Nguyen', '$2b$10$/3yhbbjXPPf3L4Z1gXDA5OJzJkf6b.2CuvIA8OzP6c8jPEQlbo5re', 4),
('Lieutenant Ryan', '$2b$10$hr1Tlo6K.yxAq3FC4iIHsuYQwYpjQC8SyDnYykMu/LNB9TXMkxMt2', 4);
```

Admiral Greer and Lieutenant Ryan still have the same passwords as before, but it's not easy to see that because of salting and hashing.

#### Create an Environment Variable
`SERVER_SESSION_SECRET` is supposed to be a secret, but right now we are pushing it to GitHub! Let's create an environment variable so that we don't do this.

* Uncomment `// return process.env.SERVER_SESSION_SECRET;` in `session-middleware.js`
* Run `npm install dotenv` to get the node module that can create environment variables
* Add the line `require('dotenv').config();` to the top of `server.js` to use the module
* add `.env` to your `.gitignore` file
* Create a `.env` file at the root of the project and paste this line into the file:
    ```
    SERVER_SESSION_SECRET=superDuperSecret
    ```
    While you're in your new `.env` file, take the time to replace `superDuperSecret` with some long random string like `25POUbVtx6RKVNWszd9ERB9Bb6` to keep your application secure. Here's a site that can help you: [https://passwordsgenerator.net/](https://passwordsgenerator.net/).
