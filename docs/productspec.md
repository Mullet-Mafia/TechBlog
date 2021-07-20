__How to write a Product Spec for dummies__
- Goal
- CUJ
- Key Decisions
- Workflow

**Goal:**
The goal, otherwise known as use-case, of the product should be a succinct message which relays the information of the end goal of the product. What does the product provide? How will the product help? Who is the product targeted for? Generally this is a short paragraph, if more than 1-2 sentences.

An example of a Goal:
```
Notify the players of a game about an on-going in-game event.
```

**Critical User Journey:**
A CUJ, or Critical User Journey, is a complex thought process that would force the author of the spec to think as a generic user. This would contain bulleted points in the process of how a user would use a tool or product. The user can even be a high-level developer, to ensure the tool will still be used correctly. Generally, you want 2-3 complex CUJs written out when designing a spec sheet. This will allow the engineer to interpret what the design is aiming for and help draft the initial demo.

An example of a CUJ:
```
As a game dev, we want to alert developers of an event in-game.
1. User will spawn into a lobby and view a physical lobby board displaying the event and a countdown.
2. User will interact with said physical board, prompting a 2D screen.
3. user will navigate the 2D screen to learn more about the event and how to participate.

As a player, I want to be alerted of an event in-game.
1. User will want a notification of the event rather than checking the lobby board.
2. User would like to be notified if any events are on-going.
3. User is prompted with a 2D screen regarding a prompt to join an event.
4. User is notified of the physical lobby board updating via a notification.
```

**Key Decisions:**
Questions to answer when writing up a product spec that should be answered by the product design. This would include any complex questions that could appear during development. Think as an end-user and the developer of the product.

An example of Key Decisions:
```
What occurs if there's too much UI clutter?
- Resort to billboard UI which can notify the user on their screen without being a permanent distraction.
How do we deal with users who are of foreign languages?
- Use icons that convey information and point the user in the right direction, using less text.
Do we want the user to close the notification?
- Yes, they should have full control of the notification system
```

**Workflows:**
The workflow will be exactly what the developer of the product uses when designing the product. This will contain information such as how the product will function and what the developer must plan for, including fall-backs if an action is already there. Aim for 2-5 workflows, 2 minimum and 5 at maximum.

Example of a workflow:
```
Launching the 2D screen:
- Clicking a button on the physical board will launch a 2D screen.
- User is prompted with a menu relatively small which contains only the necessary information, requiring no scroll.
- Information in the screen should be as follows:
[design of the product]
```