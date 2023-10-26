SpacetimeDB is a modular system intended to handle arbitrary voting algorithm registration and orchestration.

It is a realtime database meaning it can handle live vote counts, trigger side-effects, and handle vote/inputs a fast a you can send them.
(hypothetically, we could support a "twitch plays xyz" type app where user inputs are the votes.)

The DB will have a record of submissions that are to be voted on.
Each submission has an ID that points to the content server which contains the actual data, diffs, media, etc.
