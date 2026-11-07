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

Quest map pins require the separate Questie-335 addon; it is not bundled here.
