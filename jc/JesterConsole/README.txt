JesterConsole for WoW 3.3.5a

DOWNLOAD
--------
Open the GitHub Actions workflow:
https://github.com/Jestel2/JMod-WoW/actions/workflows/package-jesterconsole.yml

Choose the latest successful "Package JesterConsole addon" run for the
feature/jesterconsole-download branch, then download its JesterConsole artifact.
The artifact contains a versioned ZIP with the complete addon folder.

INSTALL / UPGRADE
-----------------
1. Extract the ZIP. It contains a JesterConsole folder.
2. Copy that folder into your WoW 3.3.5a client's Interface\AddOns directory.
   The final path must end with Interface\AddOns\JesterConsole\JesterConsole.toc.
3. Replace the existing JesterConsole folder if prompted. This does not touch
   your WoW account settings under WTF.
4. Start WoW or use /reload.

TEST
----
/jc      Open JesterConsole
/family  Open family travel
/guide   Open the quest guide

QUEST NAVIGATION
----------------
1. Install Questie-335 and TomTom for the WoW 3.3.5a client. Neither addon is
   bundled with JesterConsole.
2. Open /guide, select a quest, and click "Track + Route". The TomTom arrow
   points to Questie's nearest unfinished objective for that quest.
3. If no arrow appears, right-click the quest or an unfinished objective in
   Questie's tracker and choose "Set TomTom Target".

Questie provides quest locations; JesterConsole does not invent coordinates.
The guide shows which navigation addon is missing if the arrow tools are not ready.
