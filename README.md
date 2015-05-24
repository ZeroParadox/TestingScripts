({

     serverStartUp : function() {
         scriptChecks = 0;
         this.init();
     }
     ,
     mafiaLoad:function() {
         if (sys.existChannel("Mafia Channel")) {
             mafiachan = sys.channelId("Mafia Channel");
         } else {
             mafiachan = sys.createChannel("Mafia Channel");
         }

         shuffle=function(o){
             for(var j,x,i=o.length;i;j=parseInt(Math.random()*i),x=o[--i],o[i]=o[j],o[j]=x);
                 return o; }
         cap=function(string){
             return string.charAt(0).toUpperCase()+string.slice(1); }
         dump=function(src,mess){
             for(x in mess){
                 sys.sendMessage(src,mess[x],mafiachan); } }

   
         Config={ 
             Mafia:{
                 norepeat:3,
                 stats_file:"MafiaStats.txt"
             }
         }
         noPlayer='*';
         mafia=new function(){
             this.version="2011-07-19.2";
             var CurrentGame;
             var PreviousGames;
             var MAFIA_SAVE_FILE=Config.Mafia.stats_file;
             savePlayedGames=function(){
                 sys.writeToFile(MAFIA_SAVE_FILE,JSON.stringify(PreviousGames));
             }
             loadPlayedGames=function(){
                 try{
                     PreviousGames=JSON.parse(sys.getFileContent(MAFIA_SAVE_FILE));
                 }
                 catch(e){
                     PreviousGames=[];
                 }
             }
             loadPlayedGames();
             var defaultTheme={name:"default",sides:[{"side":"mafia","translation":"Mafia"},{"side":"mafia1","translation":"French Canadian Mafia"},{"side":"mafia2","translation":"Italian Mafia"},{"side":"village","translation":"Good people"},{"side":"werewolf","translation":"WereWolf"},{"side":"godfather","translation":"Godfather"}],roles:[{"role":"villager","translation":"Villager","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{}},{"role":"inspector","translation":"Inspector","side":"village","help":"Type /Inspect [name] to find his/her identity!","actions":{"night":{"inspect":{"target":"AnyButSelf","common":"Self","priority":30}}}},{"role":"bodyguard","translation":"Bodyguard","side":"village","help":"Type /Protect [name] to protect someone!","actions":{"night":{"protect":{"target":"AnyButSelf","common":"Role","priority":5,"broadcast":"role"}},"startup":"role-reveal"}},{"role":"mafia","translation":"Mafia","side":"mafia","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"werewolf","translation":"WereWolf","side":"werewolf","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":10}},"distract":{"mode":"ChangeTarget","hookermsg":"You tried to distract the Werewolf (what an idea, seriously), you were ravishly devoured, yum!","msg":"The ~Distracter~ came to you last night! You devoured her instead!"},"avoidHax":["kill"]}},{"role":"hooker","translation":"Pretty Lady","side":"village","help":"Type /Distract [name] to distract someone! Vote to remove people in the day!","actions":{"night":{"distract":{"target":"AnyButSelf","common":"Self","priority":1}}}},{"role":"mayor","translation":"Mayor","side":"village", "help":"You don't have any special commands during the night! Vote to remove people in the day! (your vote counts as 2)","actions":{"vote":2}},{"role":"spy","translation":"Spy","side":"village","help":"You can find out who is going to get killed next!(no command for this ability) Vote to remove people in the day!","actions":{"hax":{"kill":{"revealTeam":0.33,"revealPlayer":0.1}}}},{"role":"godfather","translation":"Godfather","side":"godfather","help":"Type /Kill [name] to kill someone! You can kill 2 targets, Type /kill [name2] again to select your second target!","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":20,"limit":2}},"distract":{"mode":"ChangeTarget","hookermsg":"You tried to seduce the Godfather... you were killed instead!","msg":"The ~Distracter~ came to you last night! You killed her instead!"},"avoidHax":["kill"]}},{"role":"vigilante","translation":"Vigilante","side":"village","help":"Type /Kill [name] to kill someone!(don't kill the good people!)","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":19}}}},{"role":"mafia1","translation":"French Canadian Mafia","side":"mafia1","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":12,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"mafia2","translation":"Italian Mafia","side":"mafia2","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"conspirator1","translation":"French Canadian Conspirator","side":"mafia1","help":"You don't have any special commands during the night! You are sided French Canadian Mafia. Vote to remove people in the day!","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"}},{"role":"conspirator2","translation":"Italian Conspirator","side":"mafia2","help":"You don't have any special commands during the night! You are sided Italian Mafia. Vote to remove people in the day!","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"}},{"role": "mafiaboss1","translation":"Don French Canadian Mafia","side":"mafia1","help":"Type /Kill [name] to kill someone! You can't be distracted!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":12,"broadcast":"team"}},"distract":{"mode":"ignore"},"startup":"team-reveal"}},{"role":"mafiaboss2","translation":"Don Italian Mafia","side":"mafia2","help":"Type /Kill [name] to kill someone! You can't be distracted!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"distract":{"mode":"ignore"},"startup":"team-reveal"}},{"role":"samurai","translation":"Samurai","side":"village","help":"Type /Kill [name] during the day phase to kill someone! You will be revealed when you kill, so make wise choices! You are allied with the Good people.","actions":{"standby":{"kill":{"target":"AnyButSelf","msg":"You can kill now using /kill [name] :","killmsg":"~Self~ pulls out a sword and strikes it through ~Target~'s chest!"}}}},{"role":"miller","translation":"Miller","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day! Oh, and insp sees you as Mafia","actions":{"inspect":{"revealAs":"mafia"}}},{"role":"truemiller","translation":"Miller","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia"},"lynch":{"revealAs":"mafia"},"startup":{"revealAs":"villager"},"onlist":"mafia"}},{ "role":"miller1","translation":"Miller","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia1"},"lynch":{"revealAs":"mafia1"},"startup":{"revealAs":"villager"},"onlist":"mafia2"}},{"role":"miller2","translation":"Miller","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia2"},"lynch":{"revealAs":"mafia2"},"startup":{"revealAs":"villager"},"onlist":"mafia1"}}],roles1:["bodyguard","mafia","inspector","werewolf","hooker","villager","truemiller","villager","mafia","villager","mayor"],roles2:["bodyguard","mafia1","mafia1","inspector","hooker","villager","mafia2","mafia2","villager","villager","villager","mayor","villager","spy","villager","miller1","miller2","mafiaboss1","villager","vigilante","villager","godfather","mafiaboss2","samurai","villager","villager","werewolf","mafia1","mafia2","bodyguard"],villagecantLoseRoles:["mayor","vigilante","samurai"]};
             var hpTheme={"villagecantLoseRoles":["mayor","hooker","samurai"],"name":"Harry Potter","roles":[{"translation":"Muggle","role":"villager","side":"village","actions":{},"help":"You have no magical powers, so all you can do is vote to remove people in the day!"},{"translation":"Harry Potter","role":"inspector","side":"village","actions":{"night":{"inspect":{"priority":30,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Inspect [name] to cast a spell to figure out a person\'s role!"},{"translation":"Hagrid","role":"bodyguard","side":"village","actions":{"startup":"role-reveal","night":{"protect":{"priority":5,"broadcast":"role","target":"AnyButSelf","common":"Role"}}},"help":"Type /Protect [name] to protect someone, you large oaf you!"},{"translation":"Snape","role":"werewolf","side":"werewolf","actions":{"vote":2,"avoidHax":["kill"],"kill":{"mode":"ignore"},"distract":{"msg":"The ~Distracter~ came to you last night! You killed the fool.","mode":"ChangeTarget","hookermsg":"You tried to distract Severus Snape (what an idea, seriously), and were promptly killed, sorry!"}},"help":"You can not be killed at night. Your vote counts as two during the day, and you are immune to the charms of the Pretty Lady. You are all alone in the world!"},{"translation":"Hermione Granger","role":"hooker","side":"village","actions":{"night":{"distract":{"priority":1,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Distract [name] at night to Stupify them! Vote to remove people in the day!"},{"translation":"Dumbledore","role":"mayor","side":"village","actions":{"vote":3},"help":"You don't have any special commands during the night. Pull your rank during the day, as your votes count as three. "},{"translation":"Argus Filch","role":"spy","side":"village","actions":{"hax":{"kill":{"revealPlayer":0.14999999999999999,"revealTeam":0.40000000000000002}}},"help":"You can find out who is going to get killed next!(no command for this ability) Vote to remove people in the day. Being superbly creepy, you can a bonus to detection over normal spies."},{"translation":"Fred Weasley","role":"mafia1","side": "mafia1","actions":{"startup":"team-reveal","night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] at night to dispose of someone!"},{"translation":"George Weasley","role":"mafia1.5","side":"mafia1","actions":{"startup":"team-reveal","night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] at night to dispose of someone!"},{"translation":"Momma Weasley","role":"mafia1.6","side":"mafia1","actions":{"startup":"team-reveal","night":{"protect":{"priority":5,"broadcast":"role","target":"AnyButSelf","common":"Role"}}},"help":"Type /Kill [name] at night to dispose of someone! You are working with Fred and George (Not Ron), and you can protect one of the two per night."},{"translation":"Death Eater","role":"mafia2","side":"mafia2","actions":{"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Head Death Eater","role":"mafia2.5","side":"mafia2","actions":{"vote":-1,"inspect":{"revealAs":"villager"},"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You are seen to the inspector as a Muggle. Your vote counts as -1. Use it to save your fellow Death Eaters!"},{"translation":"Auror","role":"samurai","side":"village","actions":{"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"~Self~ whips out his wand and points it at ~Target~, causing ~Target~ to drop dead!","target":"AnyButSelf"}}},"help":"Type /Kill [name] during the day phase to kill someone! You will be revealed when you kill, so make wise choices! You are allied with the Good people."},{"translation":"Voldemort","role":"evilsamurai","side":"mafia2","actions":{"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"Voldemort casts Avada Kedavra, and ~Target~ falls to the ground dead!","target":"AnyButSelf"}},"startup":"team-reveal"},"help":"Type /Kill [name] during the day phase to kill someone! You may only kill once per day, but you will not be revealed. You are allied with the Death Eaters."},{"translation":"Nearly Headless Nick","role":"miller","side":"village","actions":{"inspect":{"revealAs":"mafia1"}},"help":"You don't have any special commands during the night! Unfortunately, though, people are afraid of your flopping head and so the Inspector sees you as evil."},{"translation":"Peeves","role":"miller1.5","side":"village","actions":{"inspect":{"revealAs":"mafia2"}},"help":"You don't have any special commands during the night! Unfortunately, though, everyone hates you and so the Inspector sees you as evil."},{"translation":"Draco Malfoy","role":"mafia3","side":"mafia3","actions":{"startup":"team-reveal","night":{"kill":{"priority":2,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! Even the Bodyguard cannot stop you!"},{"translation":"Lucius Malfoy","role":"mafia3.5","side":"mafia3","actions":{"startup":"team-reveal","night":{"kill":{"priority":2,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! Even the Bodyguard cannot stop you!"}],"sides":[{"translation":"Weasley Brothers","side":"mafia1"},{"translation":"Death Eaters","side":"mafia2"},{"translation":"Malfoys","side":"mafia3"},{"translation":"Hogwarts","side":"village"},{"translation":"Snape","side":"werewolf"}],"roles2":["bodyguard","mafia1","mafia1.5","inspector","hooker","villager","mafia2","mafia2","villager","villager","werewolf","mayor","evilsamurai","spy","samurai","villager","miller","mafia1.6","mafia2.5","samurai","villager","villager","miller1.5","villager","villager","mafia3","mafia3.5"],"roles1":["bodyguard","mafia1","inspector","mafia1.5","hooker","villager","villager","miller","villager","mayor"]};
             var ssbbTheme={"villagecantLoseRoles":["mayor","vigilante","samurai","samus"],"name":"SSBB","roles":[{"translation":"Mario","role":"villager","side":"village","actions":{},"help":"You don't have any special commands during the night! Vote to remove people in the day!"},{"translation":"Lucas","role":"inspector","side":"village","actions":{"night":{"inspect":{"priority":30,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Inspect [name] to find his/her identity!"},{"translation":"Donkey Kong","role":"bodyguard","side":"village","actions":{"startup":"role-reveal","night":{"protect":{"priority":5,"broadcast":"role","target":"AnyButSelf","common":"Role"}}},"help":"Type /Protect [name] to protect someone!"},{"translation":"Bowser","role":"mafia","side":"mafia","actions":{"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Wolf","role":"werewolf","side":"wolf","actions":{"avoidHax":["kill"],"distract":{"msg":"The ~Distracter~ came to you last night! You devoured her instead!","mode":"ChangeTarget","hookermsg":"You tried to distract Wolf (what an idea, seriously), you were ravishly shot at!"},"night":{"kill":{"priority":10,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Peach","role":"hooker","side":"village","actions":{"night":{"distract":{"priority":1,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Distract [name] to distract someone! Vote to remove people in the day!"},{"translation":"Captain Falcon","role":"mayor","side":"village","actions":{"vote":3,"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"~Self~ pulls out a fist and punches it through ~Target~\'s chest!","target":"AnyButSelf"}}},"help":"You don't have any special commands during the night! Vote to remove people in the day! /Kill To remove people people with a falcon punch! (your vote counts as 3)"},{"translation":"Snake","role":"spy","side":"village","actions":{"hax":{"kill":{"revealPlayer":0.20000000000000001,"revealTeam":0.40000000000000002}}},"help":"You can find out who is going to get killed next!(no command for this ability) Vote to remove people in the day!"},{"translation":"Jigglypuff","role":"godfather","side":"godfather","actions":{"avoidHax":["kill"],"distract":{"msg":"The ~Distracter~ came to you last night! You killed her instead!","mode":"ChangeTarget","hookermsg":"You tried to seduce the Marshmallow, you just were rested!"},"night":{"kill":{"priority":20,"limit":2,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Kill [name] to kill someone! You can kill 2 targets, Type /kill [name2] again to select your second target!"},{"translation":"Ike","role":"vigilante","side":"village","actions":{"night":{"kill":{"priority":19,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Kill [name] to kill someone using a sword!(don't kill the good people!)"},{"translation":"Samus","role":"samus","side":"village","actions":{"distract":{"priority":1,"target":"AnyButSelf","common":"Self"},"night":{"kill":{"priority":19,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Kill [name] to kill someone using a missile! (don't kill the good people!) Type /distract to distract someone"},{"translation":"Ganondorf","role":"mafia1","side":"mafia1","actions":{"startup":"team-reveal","night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Wario","role":"mafia2","side":"mafia2","actions":{"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Waddle Doo","role":"mafia3","side":"mafia3","actions":{"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone!"},{"translation":"Moblin","role":"conspirator1","side":"mafia1","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"},"help":"You don't have any special commands during the night! You are sided Hyrulian Mafia. Vote to remove people in the day!"},{"translation":"Waluigi","role":"conspirator2","side":"mafia2","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"},"help":"You don't have any special commands during the night! You are sided Italian Mafia. Vote to remove people in the day!"},{"translation":"WaddleDee","role":"conspirator3","side":"mafia3","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"},"help":"You don't have any special commands during the night! You are sided Penguin Mafia. Vote to remove people in the day!"},{"translation":"Ganon","role":"mafiaboss1","side":"mafia1","actions":{"startup":"team-reveal","distract":{"mode":"ignore"},"night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You can\'t be distracted!"},{"translation":"Wario-Man","role":"mafiaboss2","side":"mafia2","actions":{"startup":"team-reveal","distract":{"mode":"ignore"},"night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You can\'t be distracted!"},{"translation":"King Dedede","role":"mafiaboss3","side":"mafia3","actions":{"startup":"team-reveal","distract":{"mode":"ignore"},"night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You can\'t be distracted!"},{"translation":"Marth","role":"samurai","side":"village","actions":{"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"Marth pulls out a sword and strikes it through ~Target~\'s chest!","target":"AnyButSelf"}}},"help":"Type /Kill [name] during the day phase to kill someone! You will not be revealed when you kill! You are allied with the Good people."},{"translation":"MetaKnight","role":"miller","side":"village","actions":{"inspect":{"revealAs":"mafia"}},"help":"You don't have any special commands during the night! Vote to remove people in the day! Oh, and insp sees you as Mafia"}],"sides":[{"translation":"Koopa","side":"mafia"},{"translation":"Hyrulian Mafia","side":"mafia1"},{"translation":"Italian Mafia","side":"mafia2"},{"translation":"Penguin Mafia","side":"mafia3"},{"translation":"Good people","side":"village"},{"translation":"Wolf","side":"wolf"},{"translation":"Marshmellow","side":"godfather"}],"roles2":["bodyguard","mafia1","mafia1","inspector","hooker","villager","mafia2","mafia2","villager","villager","villager","mayor","villager","spy","mafia3","mafia3","villager","mafiaboss1","villager","vigilante","samus","godfather","mafiaboss2","samurai","villager","mafiaboss3","werewolf","mafia1","mafia2","bodyguard"],"roles1":["bodyguard","mafia","inspector","werewolf","hooker","villager","mafia","villager","miller","villager","mayor"]};
             var ffTheme={"villagecantLoseRoles":["mayor","vigilante","samurai"],"name":"FF","roles":[{"translation":"Moogle","role":"villager","side":"village","actions":{},"help":"As a moogle, it is your job to get the village to win by voting during the day!"},{"translation":"Locke","role":"inspector","side":"village","actions":{"night":{"inspect":{"priority":30,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Inspect [name] to find his/her identity! Be careful to not die!"},{"translation":"Auron","role":"bodyguard","side":"village","actions":{"startup":"role-reveal","night":{"protect":{"priority":5,"broadcast":"role","target":"AnyButSelf","common":"Role"}}},"help":"Type /Protect [name] to protect someone! Try to survive!"},{"translation":"Garland","role":"mafia","side":"mafia","actions":{"startup":"team-reveal","night":{"kill":{"priority":15,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You, Garland, will knock them all down!"},{"translation":"Kefka","role":"werewolf","side":"werewolf","actions":{"avoidHax":["kill"],"distract":{"msg":"The ~Distracter~ came to you last night! You destroyed her instead!","mode":"ChangeTarget","hookermsg":"You tried to distract Kefka (how foolish...), you were killed instead!"},"night":{"kill":{"priority":3,"target":"AnyButTeam","common":"Self"}}},"help":"With insane agility, you outspeed even bodyguards. Strike hard with /kill [name]!"},{"translation":"Tifa","role":"hooker","side":"village","actions":{"night":{"distract":{"priority":2,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Distract [name] to fool around with someone! Those you mess with can do nothing!"},{"translation":"Cecil","role":"mayor","side":"village","actions":{"vote":-1},"help":"Conflicted to the core, your vote counts as -1. Use this to lead the village to victory."},{"translation":"Kuja","role":"mayor2","side":"werewolf","actions":{"vote":2},"help":"You\'re just here for a bit of fun. If you can find and partner up with Kefka, hopefully you can screw the heroes over! Your vote counts as 2."},{"translation":"Zidane","role":"spy","side":"village","actions":{"hax":{"kill":{"revealPlayer":0.14999999999999999,"revealTeam":0.40000000000000002}}},"help":"You have the ability to sense danger. Use these skills to figure out who is going to die!"},{"translation":"Sephiroth","role":"godfather","side":"godfather","actions":{"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"Sephiroth pulls out a sword and swiftly strikes it through ~Target~\'s chest!","target":"AnyButSelf"}},"avoidHax":["kill"],"distract":{"msg":"The ~Distracter~ came to you last night! You killed her instead!","mode":"ChangeTarget","hookermsg":"You tried to mess with Sephiroth, you just were killed!"},"night":{"kill":{"priority":20,"limit":1,"target":"AnyButSelf","common":"Self"}}},"help":"Type /Kill [name] to kill someone! You can kill twice, once during standby and once at night. You will not be revealed, so have fun! "},{"translation":"Lightning","role":"vigilante","side":"village","actions":{"night":{"kill":{"priority":19,"target":"AnyButSelf","common":"Self"}}},"help":"Allied with the heroes, it is your goal to assist them and win! Type /kill [name] during the night."},{"translation":"Jecht","role":"mafia1","side":"mafia1","actions":{"startup":"team-reveal","night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Team up with Seymour and Yunalesca to bring Sin to victory! Type /kill [name] during the night!"},{"translation":"Larsa Solidor","role":"mafia2","side":"mafia2","actions":{"startup":"team-reveal","night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"House Solidor unite! Use /kill [name] to weaken the heroes."},{"translation":"Yunalesca","role":"conspirator1","side":"mafia1","actions":{"startup":"team-reveal","night":{"distract":{"priority":1,"target":"AnyButSelf","common":"Self"}}},"help":"Stop those who threaten to ruin Sin\'s plans. Type /distract [name] during the night."},{"translation":"Judge Gabranth","role":"conspirator2","side":"mafia2","actions":{"startup":"team-reveal","night":{"protect":{"priority":4,"broadcast":"role","target":"AnyButSelf","common":"Role"}}},"help":"You\'re House Solidor\'s personal bodyguard. Type /protect [name] to defend Larsa or Vayne!"},{"translation":"Seymour","role":"mafiaboss1","side":"mafia1","actions":{"startup":"team-reveal","distract":{"mode":"ignore"},"night":{"kill":{"priority":12,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"Type /Kill [name] to kill someone! You can\'t be distracted by foolish mortals."},{"translation":"Vayne Solidor","role":"mafiaboss2","side":"mafia2","actions":{"startup":"team-reveal","distract":{"mode":"ignore"},"night":{"kill":{"priority":11,"broadcast":"team","target":"AnyButTeam","common":"Team"}}},"help":"As head of House Solidor, it is your duty to crush the heroes. Type /kill [name]. You cannot be distracted."},{"translation":"Cloud","role":"samurai","side":"village","actions":{"standby":{"kill":{"msg":"You can kill now using /kill [name] :","killmsg":"~Self~ pulls out a Buster Sword and strikes it through ~Target~\'s chest!","target":"AnyButSelf"}}},"help":"Lead the heroes to victory by typing /kill [name] during a standby phase! "},{"translation":"Cactuar","role":"miller","side":"village","actions":{"inspect":{"revealAs":"werewolf"}},"help":"You don't have any special commands during the night! Vote to remove people in the day! Inspector will think you\'re a bad guy!"}],"sides":[{"translation":"Garland","side":"mafia"},{"translation":"Sin","side":"mafia1"},{"translation":"House Solidor","side":"mafia2"},{"translation":"Heroes","side":"village"},{"translation":"Kefka\'s Posse","side":"werewolf"},{"translation":"Sephiroth","side":"godfather"}],"roles2":["bodyguard","mafia1","conspirator1","inspector","hooker","villager","mafia2","conspirator2","villager","miller","vigilante","mayor","werewolf","spy","villager","villager","mafia","mafiaboss1","mafiaboss2","godfather","villager","samurai","mayor2","miller","villager","villager","villager","miller","miller","mafia"],"roles1":["bodyguard","mafia","inspector","werewolf","hooker","villager","mafia","mayor","miller","villager","villager"]};
             var insanityTheme={"name":"insanity","sides":[{"side":"mafia","translation":"Green Doomguy"},{"side":"mafia1","translation":"Red Doomguy"},{"side":"mafia2","translation":"Blue Doomguy"},{"side":"village","translation":"Good team"},{"side":"werewolf","translation":"Zombie"},{"side":"godfather","translation":"Evil Boss"}],"roles":[{"role":"villager","translation":"Townie","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{}},{"role":"inspector","translation":"Scanner","side":"village","help":"Type /Inspect [name] to find his/her identity!","actions":{"night":{"inspect":{"target":"AnyButSelf","common":"Self","priority":30}}}},{"role":"bodyguard","translation":"Protectron","side":"village","help":"Type /Protect [name] to protect someone!","actions":{"night":{"protect":{"target":"AnyButSelf","common":"Role","priority":5,"broadcast":"role"}},"startup":"role-reveal"}},{"role":"mafia","translation":"Green Doomguy","side":"mafia","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"werewolf","translation":"Zombie","side":"werewolf","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":10}},"distract":{"mode":"ChangeTarget","hookermsg":"You tried to distract the Zombie (what an idea, seriously), you were ravishly devoured, yum!","msg":"The ~Music Man~ came to you last night! You devoured him instead!"},"avoidHax":["kill"]}},{"role":"hooker","translation":"Music Man","side":"village","help":"Type /Distract [name] to distract someone! Vote to remove people in the day!","actions":{"night":{"distract":{"target":"AnyButSelf","common":"Self","priority":1}}}},{"role":"mayor","translation":"Good Boss","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day! (your vote counts as 2)","actions":{"vote":2}},{"role":"spy","translation":"Camera Man","side":"village","help":"You can find out who is going to get killed next!(no command for this ability) Vote to remove people in the day!","actions":{"hax":{"kill":{"revealTeam":0.33,"revealPlayer":0.1}}}},{"role":"godfather","translation":"Evil Boss","side":"godfather","help":"Type /Kill [name] to kill someone! You can kill 2 targets, Type /kill [name2] again to select your second target!","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":20,"limit":2}},"distract":{"mode":"ChangeTarget","hookermsg":"You tried to distract the Godfather... you were killed instead!","msg":"The ~Music Man~ came to you last night! You killed him instead!"},"avoidHax":["kill"]}},{"role":"vigilante","translation":"Jon Freeman","side":"village","help":"Type /Kill [name] to kill someone!(don't kill anyone in the good team!)","actions":{"night":{"kill":{"target":"AnyButSelf","common":"Self","priority":19}}}},{"role":"mafia1","translation":"Red Doomguy","side":"mafia1","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":12,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"mafia2","translation":"Blue Doomguy","side":"mafia2","help":"Type /Kill [name] to kill someone!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"startup":"team-reveal"}},{"role":"conspirator1","translation":"Red Scientist","side":"mafia1","help":"You don't have any special commands during the night! You are sided Red Doomguy. Vote to remove people in the day!","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"}},{"role":"conspirator2","translation":"Blue Scientist","side":"mafia2","help":"You don't have any special commands during the night! You are sided Blue Doomguy. Vote to remove people in the day!","actions":{"inspect":{"revealAs":"villager"},"startup":"team-reveal"}},{"role":"mafiaboss1","translation":"Red Doomguy Boss","side":"mafia1","help":"Type /Kill [name] to kill someone! You can't be distracted!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":12,"broadcast":"team"}},"distract":{"mode":"ignore"},"startup":"team-reveal"}},{"role":"mafiaboss2","translation":"Blue Doomguy Boss","side":"mafia2","help":"Type /Kill [name] to kill someone! You can't be distracted!","actions":{"night":{"kill":{"target":"AnyButTeam","common":"Team","priority":11,"broadcast":"team"}},"distract":{"mode":"ignore"},"startup":"team-reveal"}},{"role":"samurai","translation":"Fighter","side":"village","help":"Type /Kill [name] during the day phase to kill someone! You will be revealed when you kill, so make wise choices! You are allied with the Good team.","actions":{"standby":{"kill":{"target":"AnyButSelf","msg":"You can kill now using /kill [name] :","killmsg":"~Self~ pulls out a sword and strikes it through ~Target~'s chest!"}}}},{"role":"miller","translation":"Decoy","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day! Oh, and inspector sees you as Green Doomguy.","actions":{"inspect":{"revealAs":"mafia"}}},{"role":"truemiller","translation":"Decoy","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia"},"lynch":{"revealAs":"mafia"},"startup":{"revealAs":"villager"},"onlist":"mafia"}},{"role":"miller1","translation":"Decoy","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia1"},"lynch":{"revealAs":"mafia1"},"startup":{"revealAs":"villager"},"onlist":"mafia2"}},{"role":"miller2","translation":"Decoy","side":"village","help":"You don't have any special commands during the night! Vote to remove people in the day!","actions":{"inspect":{"revealAs":"mafia2"},"lynch":{"revealAs":"mafia2"},"startup":{"revealAs":"villager"},"onlist":"mafia1"}}],"roles1":["bodyguard","mafia","inspector","werewolf","hooker","villager","truemiller","villager","mafia","villager","mayor"],"roles2":["bodyguard","mafia1","mafia1","inspector","hooker","villager","mafia2","mafia2","villager","villager","villager","mayor","villager","spy","villager","miller1","miller2","mafiaboss1","villager","vigilante","villager","godfather","mafiaboss2","samurai","villager","villager","werewolf","mafia1","mafia2","bodyguard"],"villagecantLoseRoles":["mayor","vigilante","samurai"]}
             function ThemeManager(){
                 this.themeInfo=[];
                 this.themes={};
             }
             ThemeManager.prototype.toString=function(){
                 return "[object ThemeManager]";
             }
             ThemeManager.prototype.save=function(name,url,resp){
                 var fname="mafiathemes/theme_"+name.replace("/","").toLowerCase();
                 sys.writeToFile(fname,resp);
                 var done=false;
                 for(var i=0;i<this.themeInfo.length;++i){
                     if(cmp(name,this.themeInfo[i][0])){
                         done=true;
                         this.themeInfo[i]=[name,url,fname,true];
                         break;
                     }
                 }
                 if(!done){
                     this.themeInfo.push([name,url,fname,true]);
                 }
                 sys.writeToFile("mafiathemes/metadata.json",JSON.stringify({'meta':this.themeInfo}));
             }
             ThemeManager.prototype.loadTheme=function(plain_theme){
                 var theme=new Theme();
                 try{
                     theme.sideTranslations={};
                     theme.roles={};
                     theme.nightPriority=[];
                     theme.standbyRoles=[];
                     theme.haxRoles={};
                     for(var i in plain_theme.sides){
                         theme.addSide(plain_theme.sides[i]);
                     }
                     for(var i in plain_theme.roles){
                         theme.addRole(plain_theme.roles[i]);
                     }
                     theme.roles1=plain_theme.roles1;
                     var i=2;
                     while("roles"+i in plain_theme){
                         theme["roles"+i]=plain_theme["roles"+i];++i;
                     }
                     theme.roleLists=i-1;
                     theme.villagecantLoseRoles=plain_theme.villagecantLoseRoles;
                     theme.name=plain_theme.name;
                     theme.author=plain_theme.author;
                     theme.killmsg=plain_theme.killmsg;
                     theme.killusermsg=plain_theme.killusermsg;
                     theme.generateRoleInfo();
                     theme.enabled=true;
                     return theme;
                 }
                 catch(err){
                     if(typeof sys=='object')
                         sys.sendAll("+MafiaBot: Couldn't use theme "+plain_theme.name+": "+err+".",mafiachan);
                     else
                         print("+MafiaBot: Couldn't use theme: "+plain_theme.name+": "+err+".");
                 }
             }
             ThemeManager.prototype.loadThemes=function(){
                 if(typeof sys!=="object")
                     return;
                 this.themes={};
                 this.themes["default"]=this.loadTheme(defaultTheme);
                 var content=sys.getFileContent("mafiathemes/metadata.json");
                 if(!content)
                     return;
                 var parsed=JSON.parse(content);
                 if(parsed.hasOwnProperty("meta")){
                     this.themeInfo=parsed.meta;
                 }
                 for(var i=0;i<this.themeInfo.length;++i){
                     if(!this.themeInfo[i][3])
                         continue;
                     try{
                         var theme=this.loadTheme(JSON.parse(sys.getFileContent(this.themeInfo[i][2])));
                         this.themes[theme.name.toLowerCase()]=theme;
                     }
                     catch(err){
                         sys.sendAll("+MafiaBot: Error loading cached theme \""+this.themeInfo[i][0]+"\": "+err,mafiachan);
                     }
                 }
             }
             ThemeManager.prototype.saveToFile=function(plain_theme){
                 if(typeof sys!="object")
                     return;
                 var fname="mafiathemes/theme_"+plain_theme.name.toLowerCase();
                 sys.writeToFile(fname,JSON.stringify(plain_theme));
                 this.themeInfo.push([plain_theme.name,"",fname,true]);
                 sys.writeToFile("mafiathemes/metadata.json",JSON.stringify({'meta':this.themeInfo}));
             }
             ThemeManager.prototype.loadWebTheme=function(url,announce,update){
                 if(typeof sys!='object')
                     return;
                 var manager=this;
                 sys.webCall(url,function(resp){
                                 try{
                                     var plain_theme=JSON.parse(resp);
                                     var theme=manager.loadTheme(plain_theme);
                                     var lower=theme.name.toLowerCase();
                                     if(manager.themes.hasOwnProperty(lower)&&!update){
                                         sys.sendAll("+MafiaBot: Won't update "+theme.name+" with /add, use /update to force an update",mafiachan);
                                         return;
                                     }
                                     manager.themes[lower]=theme;
                                     manager.save(theme.name,url,resp,update);
                                     if(announce){
                                         sys.sendAll("+MafiaBot: Loaded theme "+theme.name,mafiachan);
                                     }
                                 }
                                 catch(err){
                                     sys.sendAll("+MafiaBot: Couldn't download theme from "+url,mafiachan);
                                     sys.sendAll("+MafiaBot: "+err,mafiachan);
                                     return;
                                 }
                             });
             }
             ThemeManager.prototype.remove=function(src,name){
                 name=name.toLowerCase()
                 if(name in this.themes){
                     delete this.themes[name];
                     for(var i=0;i<this.themeInfo.length;++i){
                         if(cmp(name,this.themeInfo[i][0])){
                             this.themeInfo.splice(i,1);
                             break;
                         }
                     }
                     sys.writeToFile("mafiathemes/metadata.json",JSON.stringify({'meta':this.themeInfo}));
                     sys.sendAll(src,"+MafiaBot: Theme "+name+" removed.");
                 }
             }
             ThemeManager.prototype.enable=function(name){
                 name=name.toLowerCase()
                 if(name in this.themes){
                     this.themes[name].enabled=true;
                     sys.sendAll("+MafiaBot: Theme "+name+" enabled.");
                 }
             }
             ThemeManager.prototype.disable=function(name){
                 name=String(name).toLowerCase();
                 if(name in this.themes){
                     this.themes[name].enabled=false;
                     sys.sendAll("+MafiaBot: Theme "+name+" disabled.");
                 }
             }
             function Theme(){}
             Theme.prototype.toString=function(){
                 return "[object Theme]";
             }
             Theme.prototype.addSide=function(obj){
                 this.sideTranslations[obj.side]=obj.translation;
             }
             Theme.prototype.addRole=function(obj){
                 this.roles[obj.role]=obj;
                 if("hax" in obj.actions){
                     for(var i in obj.actions.hax){
                         var action=i;
                         if(!(action in this.haxRoles)){
                             this.haxRoles[action]=[];
                         }
                         this.haxRoles[action].push(obj.role);
                     }
                 }
                 if("night" in obj.actions){
                     for(var i in obj.actions.night){
                         var priority=obj.actions.night[i].priority;
                         var action=i;
                         var role=obj.role;
                         this.nightPriority.push({'priority':priority,'action':action,'role':role});
                     }
                     this.nightPriority.sort(function(a,b){
                                                 return a.priority-b.priority
                                             });
                 }
                 if("standby" in obj.actions){
                     for(var i in obj.actions.standby){
                         this.standbyRoles.push(obj.role);
                     }
                 }
             }
             Theme.prototype.generateRoleInfo=function(){
                 var sep="*** *********************************************************************** ***";
                 var roles=[sep];
                 for(var r in this.roles){
                     try{
                         var role=this.roles[r];
                         roles.push("±Role: "+role.translation);
                         var abilities="";
                         if(role.actions.night){
                             for(var a in role.actions.night){
                                 var ability=role.actions.night[a];
                                 abilities+="Can "+a+" "+("limit" in ability?ability.limit+" persons":"one person")+" during the night. ";
                                 if("avoidHax" in role.actions&&role.actions.avoidHax.indexOf(a)!=-1){
                                     abilities+="(Can't be detected by spies.) ";
                                 }
                             }
                         }
                         if(role.actions.standby){
                             for(var a in role.actions.standby){
                                 var ability=role.actions.standby[a];
                                 abilities+="Can "+a+" "+("limit" in ability?ability.limit+" persons":"one person")+" during the standby. ";
                             }
                         }
                         if("vote" in role.actions){
                             abilities+="Vote counts as "+role.actions.vote+". ";
                         }
                         if("kill" in role.actions&&role.actions.kill.mode=="ignore"){
                             abilities+="Can't be nightkilled. ";
                         }
                         if("hax" in role.actions&&Object.keys){
                             var haxy=Object.keys(role.actions.hax);
                             abilities+="Gets hax on "+(haxy.length>1?haxy.splice(0,haxy.length-1).join(", ")+" and ":"")+haxy+". ";
                         }
                         if("inspect" in role.actions){
                             abilities+="Reveals as "+this.roles[role.actions.inspect.revealAs].translation+" when inspected. ";
                         }
                         if("distract" in role.actions){
                             if(role.actions.distract.mode=="ChangeTarget")
                                 abilities+="Kills any distractors. ";
                             if(role.actions.distract.mode=="ignore")
                                 abilities+="Ignores any distractors. ";
                         }
                         if(typeof role.side=="string"){
                             abilities+="Sided with "+this.trside(role.side)+". ";
                         }
                         else if(typeof role.side=="object"){
                             var plop=Object.keys(role.side.random);
                             var tran=[];
                             for(var p in plop){
                                 tran.push(this.trside(p));
                             }
                             abilities+="Sided with "+(tran.length>1?tran.splice(0,tran.length-1).join(", ")+" or ":"")+tran+". ";
                         }
                         roles.push("±Ability: "+abilities);
                         var parts=[];
                         var end=0;
                         for(var i=1;i<=this.roleLists;++i){
                             var r="roles"+i;
                             var start=this[r].indexOf(role.role);
                             var last=end;
                             end=this[r].length;
                             if(start>=0){
                                 ++start;
                                 start=start>last?start:1+last;
                                 if(parts.length>0&&parts[parts.length-1][1]==start-1){
                                     parts[parts.length-1][1]=end;
                                 }
                                 else{
                                     parts.push([start,end]);
                                     if(parts.length>1){
                                         parts[parts.length-2]=parts[parts.length-2][0]<parts[parts.length-2][1]?parts[parts.length-2].join("-"):parts[parts.length-2][1];
                                     }
                                 }
                             }
                         }
                         if(parts.length>0){
                             parts[parts.length-1]=parts[parts.length-1][0]<parts[parts.length-1][1]?parts[parts.length-1].join("-"):parts[parts.length-1][1];
                         }
                         roles.push("±Game: "+parts.join(", ")+" Players");
                         roles.push(sep);
                     }
                     catch(err){
                         sys.sendAll("+MafiaBot: Error adding role "+role.translation+"("+role.role+") to /roles",mafiachan);
                         throw err;
                     }
                 }
                 this.roleInfo=roles;
             }
             Theme.prototype.trside=function(side){
                 return this.sideTranslations[side];
             }
             Theme.prototype.trrole=function(role){
                 return this.roles[role].translation;
             }
             Theme.prototype.getHaxRolesFor=function(command){
                 if(command in this.haxRoles){
                     return this.haxRoles[command];
                 }
                 return[];
             }
             this.isInGame=function(player){
                 if(mafia.state=="entry"){
                     return this.signups.indexOf(player)!=-1;
                 }
                 return player in this.players;
             };
             this.themeManager=new ThemeManager();
             this.hasCommand=function(name,command,state){
                 var player=this.players[name];
                 return(state in player.role.actions&&command in player.role.actions[state]);
             };
             this.correctCase=function(string){
                 var lstring=string.toLowerCase();
                 for(var x in this.players){
                     if(x.toLowerCase()==lstring)
                         return this.players[x].name;
                 }
                 return noPlayer;
             };
             this.clearVariables=function(){
                 this.players={};
                 this.signups=[];
                 this.state="blank";
                 this.ticks=0;
                 this.votes={};
                 this.voteCount=0;
                 this.ips=[];
                 this.resetTargets();
             };
             this.lastAdvertise=0;
             this.resetTargets=function(){
                 this.teamTargets={};
                 this.roleTargets={};
                 for(var p in this.players){
                     this.players[p].targets={};
                     this.players[p].dayKill=undefined;
                     this.players[p].guarded=undefined;
                 }
             };
             this.clearVariables();
             this.startGame = function(src, commandData) {
                 if (mafia.state != "blank") {
                     sys.sendMessage(src, "±Game: A game is going on. Wait until it's finished to start another one", mafiachan);
                     sys.sendMessage(src, "±Game: You can join the game by typing /join!", mafiachan);
                     return;
                 }

                 var previous = mafia.theme ? mafia.theme.name : undefined;
                 var themeName = commandData == noPlayer ? "default" : commandData.toLowerCase();
                 // games need to go default, theme, default, theme...
                 if (sys.auth(src) < 1 && previous !== undefined) {
                     if (previous == "default" && themeName == "default") {
                         sys.sendMessage(src, "±Game: Can't repeat normal game!", mafiachan);
                         return;
                     }
                     else if (previous !== "default" && themeName !== "default") {
                         sys.sendMessage(src, "±Game: Can't repeat themed game!", mafiachan);
                         return;
                     }
                 }
                 try {
                     var Check = PreviousGames.slice(-Config.Mafia.norepeat).reverse();
                     for (var i = 0; i < Check.length; ++i) {
                         if (Check[i].what == themeName && themeName != "default") {
                             sys.sendMessage(src, "±Game: This was just played " + i + " games ago, no repeating!", mafiachan);
                             return;
                         }
                     }
		 }
		 catch(e) {
		     // Do nothing //
		 }
		 if(themeName == "random") {
		     mafia.theme = mafia.themeManager.themes[Object.keys(mafia.themeManager.themes)[parseInt(Object.keys(mafia.themeManager.themes).length * Math.random())]];
                 }
                 if (themeName in mafia.themeManager.themes&&themeName != "random") {
                     if (!mafia.themeManager.themes[themeName].enabled) {
                         sys.sendMessage(src, "±Game: This theme is disabled!", mafiachan);
                         return;
                     }
                     mafia.theme = mafia.themeManager.themes[themeName];
                 } else {
                     sys.sendMessage(src, "±Game: No such theme!", mafiachan);
                     return;
                 }

                 CurrentGame = {who: sys.name(src), what: mafia.theme, when: parseInt(sys.time()), playerCount: 0};
                 
                 // For random theme
                 //mafia.theme = mafia.themeManager.themes[Object.keys(mafia.themeManager.themes)[parseInt(Object.keys(mafia.themeManager.themes).length * Math.random())]];
                 

                 sys.sendAll("", mafiachan);
                 sys.sendAll("*** ************************************************************************************", mafiachan);
                 if (mafia.theme.name == "default") {
                     sys.sendAll("±Game: " + sys.name(src) + " started a game!", mafiachan);
                 } else {
                     sys.sendAll("±Game: " + sys.name(src) + " started a game with theme "+mafia.theme.name+"!", mafiachan);
                 }
                 sys.sendAll("±Game: Type /Join to enter the game!", mafiachan);
                 sys.sendAll("*** ************************************************************************************", mafiachan);
                 sys.sendAll("", mafiachan);

                 if (sys.playersOfChannel(mafiachan).length < 20) {
                     var time = parseInt(sys.time());
                     if (time > mafia.lastAdvertise + 60*15) {
                         mafia.lastAdvertise = time;
                         sys.sendAll("", 0);
                         sys.sendAll("*** ************************************************************************************", 0);
                         sys.sendAll("±Game: " + sys.name(src) + " started a mafia game!", 0);
                         sys.sendAll("±Game: Go in the #Mafia Channel and type /Join to enter the game!", 0);
                         sys.sendAll("*** ************************************************************************************", 0);
                         sys.sendAll("", 0);
                     }
                 }
                 mafia.clearVariables();
                 mafia.state = "entry";

                 mafia.ticks = 60;
             };
	     
             this.endGame=function(src){
                 if(mafia.state=="blank"){
                     sys.sendMessage(src,"±Game: No game is going on.",mafiachan);
                     return;
                 }
                 sys.sendAll("*** ************************************************************************************",mafiachan);
                 sys.sendAll("±Game: "+(src?sys.name(src):"+MafiaBot")+" has stopped the game!",mafiachan);
                 sys.sendAll("*** ************************************************************************************",mafiachan);
                 sys.sendAll("",mafiachan);
                 mafia.clearVariables();
             };
             this.tickDown=function(){
                 if(this.ticks<=0){
                     return;
                 }
                 this.ticks=this.ticks-1;
                 if(this.ticks==0)
                     this.callHandler(this.state);
                 else{
                     if (this.ticks == 30 && this.state == "entry") {
                         sys.sendAll("",mafiachan);
                         sys.sendAll("±Game: Hurry up, you only have "+this.ticks+" more seconds to join!", mafiachan);
                         sys.sendAll("",mafiachan);
                     }
                 }
             };
             this.sendPlayer=function(player,message){
                 var id=sys.id(player);
                 if(id==undefined)
                     return;
                 sys.sendMessage(id,message,mafiachan);
             };
             this.getPlayersForTeam=function(side){
                 var team=[];
                 for(var p in this.players){
                     var player=this.players[p];
                     if(player.role.side==side){
                         team.push(player.name);
                     }
                 }
                 return team;
             };
             this.getPlayersForTeamS=function(side){
                 return mafia.getPlayersForTeam(side).join(", ");
             };
             this.getPlayersForRole=function(role){
                 var team=[]
                 for(var p in this.players){
                     var player=this.players[p];
                     if(player.role.role==role){
                         team.push(player.name);
                     }
                 }
                 return team;
             };
             this.getPlayersForRoleS=function(role){
                 return mafia.getPlayersForRole(role).join(", ");
             };
             this.getCurrentRoles=function(){
                 var list=[]
                 for(var p in this.players){
                     if(typeof this.players[p].role.actions.onlist==="string")
                         list.push(this.theme.trrole(this.players[p].role.actions.onlist));
                     else
                         list.push(this.players[p].role.translation);
                 }
                 return list.sort().join(", ");
             };
             this.getCurrentPlayers=function(){
                 var list=[];
                 for(var p in this.players){
                     list.push(this.players[p].name);
                 }
                 return list.sort().join(", ");
             }
             this.player=function(role){
                 for(var p in this.players){
                     if(mafia.players[p].role.role==role)
                         return x;
                 }
                 return noPlayer;
             };
             this.removePlayer=function(player){
                 for(var action in player.role.actions.night){
                     var targetMode=player.role.actions.night[action].target;
                     var team=this.getPlayersForTeam(player.role.side);
                     var role=this.getPlayersForRole(player.role.role);
                     if((targetMode=='AnyButSelf'||targetMode=='Any')||(targetMode=='AnyButTeam'&&team.length==1)||(targetMode=='AnyButRole'&&role.length==1)){
                         this.removeTarget(player,action);
                     }
                 }
                 if(mafia.votes.hasOwnProperty(player.name))
                     delete mafia.votes[player.name];
                 delete this.players[player.name];
             };
             this.kill=function(player){
                 if(this.theme.killmsg){
                     sys.sendAll(this.theme.killmsg.replace(/~Player~/g,player.name).replace(/~Role~/g,player.role.translation),mafiachan);
                 }
                 else{
                     sys.sendAll("±Kill: "+player.name+" ("+player.role.translation+") died!",mafiachan);
                 }
                 this.removePlayer(player);
             };
             this.removeTargets=function(player){
                 for(var action in player.role.actions.night){
                     this.removeTarget(player,action);
                 }
             };
             this.removeTarget=function(player,action){
                 var targetMode=player.role.actions.night[action].common;
                 if(targetMode=='Self'){
                     player.targets[action]=[];
                 }
                 else if(targetMode=='Team'){
                     if(!(player.role.side in this.teamTargets)){
                         this.teamTargets[player.role.side]={};
                     }
                     this.teamTargets[player.role.side][action]=[];
                 }
                 else if(targetMode=='Role'){
                     if(!(player.role.role in this.roleTargets)){
                         this.roleTargets[player.role.role]={};
                     }
                     this.roleTargets[player.role.role][action]=[];
                 }
             };
             this.removeTarget2=function(player,target){
             };
             this.getTargetsFor=function(player,action){
                 var commonTarget=player.role.actions.night[action].common;
                 if(commonTarget=='Self'){
                     if(!(action in player.targets)){
                         player.targets[action]=[];
                     }
                     return player.targets[action];
                 }
                 else if(commonTarget=='Team'){
                     if(!(player.role.side in this.teamTargets)){
                         this.teamTargets[player.role.side]={};
                     }
                     if(!(action in this.teamTargets[player.role.side])){
                         this.teamTargets[player.role.side][action]=[];
                     }
                     return this.teamTargets[player.role.side][action];
                 }
                 else if(commonTarget=='Role'){
                     if(!(player.role.role in this.roleTargets)){
                         this.roleTargets[player.role.role]={};
                     }
                     if(!(action in this.roleTargets[player.role.role])){
                         this.roleTargets[player.role.role][action]=[];
                     }
                     return this.roleTargets[player.role.role][action];
                 }
             };
             this.setTarget=function(player,target,action){
                 var commonTarget=player.role.actions.night[action].common;
                 var limit=1;
                 if(player.role.actions.night[action].limit!==undefined){
                     limit=player.role.actions.night[action].limit;
                 }
                 var list;if(commonTarget=='Self'){
                     if(!(action in player.targets)){
                         player.targets[action]=[];
                     }
                     list=player.targets[action];
                 }
                 else if(commonTarget=='Team'){
                     if(!(player.role.side in this.teamTargets)){
                         this.teamTargets[player.role.side]={};
                     }
                     if(!(action in this.teamTargets[player.role.side])){
                         this.teamTargets[player.role.side][action]=[];
                     }
                     list=this.teamTargets[player.role.side][action];
                 }
                 else if(commonTarget=='Role'){
                     if(!(player.role.role in this.roleTargets)){
                         this.roleTargets[player.role.role]={};
                     }
                     if(!(action in this.roleTargets[player.role.role])){
                         this.roleTargets[player.role.role][action]=[];
                     }
                     list=this.roleTargets[player.role.role][action];
                 }
                 if(list.indexOf(target.name)==-1){
                     list.push(target.name);
                     if(list.length>limit){
                         list.splice(0,1);
                     }
                 }
                 if(this.ticks>0&&limit>1)
                     this.sendPlayer(player.name,"±Game: Your target(s) are "+list.join(', ')+"!");
             };
             this.testWin=function(){
                 if(Object.keys(mafia.players).length==0){
                     sys.sendAll("±Game: Everybody died! This is why we can't have nice things :(",mafiachan);
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     mafia.clearVariables();
                     return true;
                 }
                 outer:
                 for(var p in mafia.players){
                     var winSide=mafia.players[p].role.side;
                     if(winSide!='village'){
                         for(var i in mafia.theme.villagecantLoseRoles){
                             if(mafia.player(mafia.theme.villagecantLoseRoles[i])!=noPlayer)
                                 continue outer;
                         }
                     }
                     var players=[];
                     var goodPeople=[];
                     for(var x in mafia.players){
                         if(mafia.players[x].role.side==winSide){
                             players.push(x);
                         }
                         else if(winSide=='village'){
                             continue outer;
                         }
                         else if(mafia.players[x].role.side=='village'){
                             goodPeople.push(x);
                         }
                         else{
                             return false;
                         }
                     }
                     if(players.length>=goodPeople.length){
                         sys.sendAll("±Game: The "+mafia.theme.trside(winSide)+" ("+players.join(', ')+") wins!",mafiachan);
                         if(goodPeople.length>0){
                             sys.sendAll("±Game: The "+mafia.theme.trside('village')+" ("+goodPeople.join(', ')+") lose!",mafiachan);
                         }
                         sys.sendAll("*** ************************************************************************************",mafiachan);
                         mafia.clearVariables();
                         return true;
                     }
                 }
                 return false;
             };
             this.handlers={
                 entry:function(){
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     sys.sendAll("Time Over! :",mafiachan);
                     try {
                         CurrentGame.playerCount=mafia.signups.length;
                         PreviousGames.push(CurrentGame);
                         savePlayedGames();
                     }
                     catch(e) {}
                     if(mafia.signups.length<5){
                         sys.sendAll("Insufficient amount of players! :",mafiachan);
                         sys.sendAll("You need at least 5 players to join (Total; "+mafia.signups.length+").",mafiachan);
                         sys.sendAll("*** ************************************************************************************",mafiachan);
                         mafia.clearVariables();
                         return;
                     }
                     var i=1;
                     while(mafia.signups.length>mafia.theme["roles"+i].length){
                         ++i;
                     }
                     var srcArray=mafia.theme["roles"+i].slice(0,mafia.signups.length);
                     srcArray=shuffle(srcArray);
                     for(var i=0;i<srcArray.length;++i){
                         mafia.players[mafia.signups[i]]={
                             'name':mafia.signups[i],
                             'role':mafia.theme.roles[srcArray[i]],
                             'targets':{}
                         };
                         if(typeof mafia.theme.roles[srcArray[i]].side=="object"){
                             if("random" in mafia.theme.roles[srcArray[i]].side){
                                 var cum=0;
                                 var val=sys.rand(1,100)/100;
                                 for(var side in mafia.theme.roles[srcArray[i]].side.random){
                                     cum+=mafia.theme.roles[srcArray[i]].side.random[side];
                                     if(cum>=val){
                                         mafia.players[mafia.signups[i]].role.side=side;
                                         break;
                                     }
                                 }
                                 if(typeof mafia.players[mafia.signups[i]].role.side=="object"){
                                     sys.sendAll("+MafiaBot: Broken theme...",mafiachan);
                                     return;
                                 }
                             }
                             else{
                                 sys.sendAll("+MafiaBot: Broken theme...",mafiachan);
                                 return;
                             }
                         }
                     }
                     sys.sendAll("The Roles have been decided! :",mafiachan);
                     for(var p in mafia.players){
                         var player=mafia.players[p];
                         var role=player.role;
                         if(typeof role.actions.startup=="object"&&typeof role.actions.startup.revealAs=="string"){
                             mafia.sendPlayer(player.name,"±Game: Your role is "+mafia.theme.trrole(role.actions.startup.revealAs)+"!");
                         }
                         else{
                             mafia.sendPlayer(player.name,"±Game: Your role is "+role.translation+"!");}
                         mafia.sendPlayer(player.name,"±Game: "+role.help);
                         if(role.actions.startup=="team-reveal"){
                             mafia.sendPlayer(player.name,"±Game: Your team is "+mafia.getPlayersForTeamS(role.side)+".");
                         }
                         if(role.actions.startup=="team-revealif"){
                             if(role.actions.startup["team-revealif"].indexOf(role.side)!=-1){
                                 mafia.sendPlayer(player.name,"±Game: Your team is "+mafia.getPlayersForTeamS(role.side)+".");
                             }
                         }
                         if(role.actions.startup=="role-reveal"){
                             mafia.sendPlayer(player.name,"±Game: People with your role are "+mafia.getPlayersForRoleS(role.role)+".");
                         }
                         if(typeof role.actions.startup=="object"&&role.actions.startup.revealRole){
                             mafia.sendPlayer(player.name,"±Game: The "+mafia.theme.roles[role.actions.startup.revealRole].translation+" is "+mafia.getPlayersForRoleS(player.role.actions.startup.revealRole)+"!");
                         }
                     }
                     sys.sendAll("Current Roles: "+mafia.getCurrentRoles()+".",mafiachan);
                     sys.sendAll("Current Players: "+mafia.getCurrentPlayers()+".",mafiachan);
                     sys.sendAll("Time: Night",mafiachan);
                     sys.sendAll("Make your moves, you only have 30 seconds! :",mafiachan);
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     mafia.ticks=30;
                     mafia.state="night";
                     mafia.resetTargets();
                 }
                 ,
                 night:function(){
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     sys.sendAll("Time Over! :",mafiachan);
                     var nightkill=false;
                     var getTeam=function(role,commonTarget){
                         var team=[];
                         if(commonTarget=='Role'){
                             team=mafia.getPlayersForRole(role.role);
                         }
                         else if(commonTarget=='Team'){
                             team=mafia.getPlayersForTeam(role.side);
                         }
                         return team;
                     }
                     for(var i in mafia.theme.nightPriority){
                         var o=mafia.theme.nightPriority[i];
                         var names=mafia.getPlayersForRole(o.role);
                         var command=o.action;
                         if("command" in mafia.theme.roles[o.role].actions.night[o.action]){
                             command=mafia.theme.roles[o.role].actions.night[o.action].command;
                         }
                         for(var j=0;j<names.length;++j){
                             if(!mafia.isInGame(names[j]))
                                 continue;
                             var player=mafia.players[names[j]];
                             var targets=mafia.getTargetsFor(player,o.action);
                             if(command=="distract"){
                                 if(targets.length==0)
                                     continue;
                                 var target=targets[0];
                                 if(!mafia.isInGame(target))
                                     continue;
                                 target=mafia.players[target];
                                 var distractMode=target.role.actions.distract;
                                 if(distractMode===undefined){
                                 }
                                 else if(distractMode.mode=="ChangeTarget"){
                                     mafia.sendPlayer(player.name,"±Game: "+distractMode.hookermsg);
                                     mafia.sendPlayer(target.name,"±Game: "+distractMode.msg.replace(/~Distracter~/g,player.role.translation));
                                     mafia.kill(player);
                                     nightkill=true;
                                     mafia.removeTargets(target);
                                     continue;
                                 }
                                 else if(distractMode.mode=="ignore"){
                                     mafia.sendPlayer(target.name,"±Game: "+distractMode.msg);
                                     continue;
                                 }
                                 else if(typeof distractMode.mode=="object"&&(typeof distractMode.mode.ignore=="string"&&distractMode.mode.ignore==player.role.role||typeof distractMode.mode.ignore=="object"&&typeof distractMode.mode.ignore.indexOf=="function"&&distractMode.mode.ignore.indexOf(player.role.role)>-1)){
                                     if(distractMode.msg)
                                         mafia.sendPlayer(target.name,"±Game: "+distractMode.msg);
                                     continue;
                                 }
                                 else if(typeof distractMode.mode=="object"&&typeof distractMode.mode.killif=="object"&&typeof distractMode.mode.killif.indexOf=="function"&&distractMode.mode.killif.indexOf(player.role.role)>-1){
                                     if(distractMode.hookermsg)
                                         mafia.sendPlayer(player.name,"±Game: "+distractMode.hookermsg);
                                     if(distractMode.msg)
                                         mafia.sendPlayer(target.name,"±Game: "+distractMode.msg.replace(/~Distracter~/g,player.role.translation));
                                     mafia.kill(player);
                                     nightkill=true;
                                     mafia.removeTargets(target);
                                     continue;
                                 }
                                 mafia.sendPlayer(target.name,"±Game: The "+player.role.translation+" came to you last night! You were too busy being distracted!");
                                 mafia.removeTargets(target);
                                 if("night" in target.role.actions){
                                     for(var action in target.role.actions.night){
                                         var team=getTeam(target.role,target.role.actions.night[action].common);
                                         for(var x in team){
                                             if(team[x]!=target.name){
                                                 mafia.sendPlayer(team[x],"±Game: Your teammate was too busy with the "+player.role.translation+" during the night, you decided not to "+action+" anyone during the night!");
                                             }
                                         }
                                     }
                                 }
                             }

                             else if(command=="protect"){
                                 for(var t in targets){
                                     var target=targets[t];
                                     if(mafia.isInGame(target)){
                                         mafia.players[target].guarded=true;
                                     }
                                 }
                             }
                             else if(command=="inspect"){
                                 if(targets.length==0)
                                     continue;
                                 var target=targets[0];
                                 if(!mafia.isInGame(target))
                                     continue;
                                 target=mafia.players[target];
                                 var inspectMode=target.role.actions.inspect;
                                 if(inspectMode===undefined){
                                     mafia.sendPlayer(player.name,"±Info: "+target.name+" is the "+target.role.translation+"!!");
                                 }
                                 else{
                                     if(inspectMode.revealAs!==undefined){
                                         mafia.sendPlayer(player.name,"±Info: "+target.name+" is the "+mafia.theme.trrole(inspectMode.revealAs)+"!!");
                                     }
                                     if(inspectMode.revealSide!==undefined){
                                         mafia.sendPlayer(player.name,"±Info: "+target.name+" is sided with the "+mafia.theme.trside(target.role.side)+"!!");
                                     }
                                 }
                             }
                             else if(command=="poison"){
                                 for(var t in targets){
                                     var target=targets[t];
                                     if(!mafia.isInGame(target))
                                         continue;
                                     target=mafia.players[target];
                                     if(target.poisoned==undefined){
                                         target.poisoned= 1;
                                     }
                                 }
                             }
                             else if(command=="kill"){
                                 for(var t in targets){
                                     var target=targets[t];
                                     if(!mafia.isInGame(target))
                                         continue;
                                     target=mafia.players[target];
                                     var revenge=false;
                                     var revengetext="±Game: You were killed during the night!";
                                     if("kill" in target.role.actions&&target.role.actions.kill.mode=="killattacker"){
                                         revenge=true;
                                         if(target.role.actions.kill.msg)
                                             revengetext=target.role.actions.kill.msg;
                                     }
                                     if(target.guarded){
                                         mafia.sendPlayer(player.name,"±Game: Your target ("+target.name+") was protected!");
                                     }
                                     else if("kill" in target.role.actions&&target.role.actions.kill.mode=="ignore"){
                                         mafia.sendPlayer(player.name,"±Game: Your target ("+target.name+") evaded the kill!");
                                     }
                                     else if("kill" in target.role.actions&&typeof target.role.actions.kill.mode=="object"&&typeof target.role.actions.kill.mode.evadeChance<sys.rand(1,100)/100){
                                         mafia.sendPlayer(player.name,"±Game: Your target ("+target.name+") evaded the kill!");
                                     }
                                     else{
                                         if(mafia.theme.killusermsg){
                                             mafia.sendPlayer(target.name,mafia.theme.killusermsg);
                                         }
                                         else{
                                             mafia.sendPlayer(target.name,"±Game: You were killed during the night!");
                                         }
                                         mafia.kill(target); 
                                         nightkill=true;
                                     }
                                     if(revenge){
                                         mafia.sendPlayer(player.name,revengetext);
                                         mafia.kill(player);
                                         nightkill=true;
                                     }
                                 }
                             }
                         }
                     }
                     for(var p in mafia.players){
                         var player=mafia.players[p];
                         if(player.poisoned==1){
                             player.poisoned=2;
                         }
                         else if(player.poisoned==2){
                             mafia.sendPlayer(target.name,"±Game: You died because of poison!");
                             mafia.kill(player);
                             nightkill=true;
                         }
                     }
                     if(!nightkill){
                         sys.sendAll("No one died! :",mafiachan);
                     }
                     if(mafia.testWin()){
                         return;
                     }
                     mafia.ticks=30;
                     if(mafia.players.length>=15){
                         mafia.ticks=40;
                     }
                     else if(mafia.players.length<=4){
                         mafia.ticks=15;
                     }
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     sys.sendAll("Current Roles: "+mafia.getCurrentRoles()+".",mafiachan);
                     sys.sendAll("Current Players: "+mafia.getCurrentPlayers()+".",mafiachan);
                     sys.sendAll("Time: Day",mafiachan);
                     sys.sendAll("You have "+mafia.ticks+" seconds to debate who are the bad guys! :",mafiachan);
                     for(var role in mafia.theme.standbyRoles){
                         var names=mafia.getPlayersForRole(mafia.theme.standbyRoles[role]);
                         for(var j=0;j<names.length;++j){
                             for(var k in mafia.players[names[j]].role.actions.standby){
                                 mafia.sendPlayer(names[j],mafia.players[names[j]].role.actions.standby[k].msg);
                             }
                         }
                     }
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     mafia.state="standby";
                 }
                 ,
                 standby:function(){
                     mafia.ticks=30;
                     sys.sendAll("*** ************************************************************************************",mafiachan);sys.sendAll("Current Roles: "+mafia.getCurrentRoles()+".",mafiachan);
                     sys.sendAll("Current Players: "+mafia.getCurrentPlayers()+".",mafiachan);
                     sys.sendAll("Time: Day",mafiachan);
                     sys.sendAll("It's time to vote someone off, type /Vote [name], you only have "+mafia.ticks+" seconds! :",mafiachan);
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     mafia.state="day";
                     mafia.votes={};
                     mafia.voteCount=0;
                 }
                 ,
                 day: function(){
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     sys.sendAll("Time Over! :",mafiachan);
                     var voted={};
                     for(var pname in mafia.votes){
                         var player=mafia.players[pname];var target=mafia.votes[pname];
                         if(!(target in voted)){
                             voted[target]=0;
                         }
                         if(player.role.actions.vote!==undefined){
                             voted[target]+=player.role.actions.vote;
                         }
                         else{
                             voted[target]+=1;}
                     }

                     var tie=true;
                     var maxi=0;
                     var downed=noPlayer;
                     for(var x in voted){
                         if(voted[x]==maxi){
                             tie=true;
                         }
                         else if(voted[x]>maxi){
                             tie=false;
                             maxi=voted[x];
                             downed=x;
                         }
                     }
                     if(tie){
                         sys.sendAll("No one was voted off! :",mafiachan);
                         sys.sendAll("*** ************************************************************************************",mafiachan);
                     }
                     else{
                         var roleName=typeof mafia.players[downed].role.actions.lynch=="object"&&typeof mafia.players[downed].role.actions.lynch.revealAs=="string"?mafia.theme.trrole(mafia.players[downed].role.actions.lynch.revealAs):mafia.players[downed].role.translation
                         sys.sendAll("±Game: "+downed+" ("+roleName+") was removed from the game!",mafiachan);
                         mafia.removePlayer(mafia.players[downed]);
                         if(mafia.testWin())
                             return;
                     }
                     sys.sendAll("Current Roles: "+mafia.getCurrentRoles()+".",mafiachan);
                     sys.sendAll("Current Players: "+mafia.getCurrentPlayers()+".",mafiachan);
                     sys.sendAll("Time: Night",mafiachan);
                     sys.sendAll("Make your moves, you only have 30 seconds! :",mafiachan);
                     sys.sendAll("*** ************************************************************************************",mafiachan);
                     mafia.ticks=30;
                     mafia.state="night";
                     mafia.resetTargets();
                 }
             };
             this.callHandler=function(state){
                 if(state in this.handlers)
                     this.handlers[state]();
             };
             this.showCommands=function(src){
                 sys.sendMessage(src,"",mafiachan);
                 sys.sendMessage(src,"Server Commands:",mafiachan);
                 for(x in mafia.commands["user"]){
                     sys.sendMessage(src,"/"+cap(x)+" - "+mafia.commands["user"][x][1],mafiachan);
                 }
                 if(sys.auth(src)>0||mafia.isMafiaAdmin(src)){
                     sys.sendMessage(src,"Authority Commands:",mafiachan);
                     for(x in mafia.commands["auth"]){
                         sys.sendMessage(src,"/"+cap(x)+" - "+mafia.commands["auth"][x][1],mafiachan);
                     }
                 }
                 sys.sendMessage(src,"",mafiachan);
             };
             this.showHelp=function(src){
                 var help=[
                     "*** *********************************************************************** ***",
                     "±Game: The objective in this game on how to win depends on the role you are given.",
                     "*** *********************************************************************** ***",
                     "±Role: Mafia",
                     "±Win: Eliminate the WereWolf and the Good People!",
                     "*** *********************************************************************** ***",
                     "±Role: WereWolf",
                     "±Win: Eliminate everyone else in the game!",
                     "*** *********************************************************************** ***",
                     "±Role: Good people (Inspector, Bodyguard, Pretty Lady, Villager, Mayor, Spy, Vigilante, Samurai, Miller)",
                     "±Win: Eliminate the WereWolf, Mafia (French and Italian if exists) and the Godfather!",
                     "*** *********************************************************************** ***",
                     "±Role: French Canadian Mafia, Don French Canadian Mafia",
                     "±Win: Eliminate the Italian Mafia, Godfather and the Good People!",
                     "*** *********************************************************************** ***",
                     "±Role: Italian Mafia, Don Italian Mafia",
                     "±Win: Eliminate the French Canadian Mafia, Godfather and the Good People!",
                     "*** *********************************************************************** ***",
                     "±More: Type /roles for more info on the characters in the game!",
                                         "±More: Type /rules to see some rules you should follow during a game!",
                                         "*** *********************************************************************** ***",
                                         ""];
                 dump(src,help);
             };
             this.showRoles=function(src,commandData){
                 var themeName="default";
                 if(mafia.state!="blank"){
                     themeName=mafia.theme.name.toLowerCase();
                 }
                 if(commandData!=noPlayer){
                     themeName=commandData.toLowerCase();
                     if(!mafia.themeManager.themes.hasOwnProperty(themeName)){
                         sys.sendMessage(src,"±Game: No such theme!",mafiachan);
                         return;
                     }
                 }
                 roles=mafia.themeManager.themes[themeName].roleInfo;
                 dump(src,roles);
             };
             this.showRules=function(src){
                 var rules=["",
                            "±Rule: All rules apply.", 
                            "±Hint: Use /rules for these rules.",
                                               "",
                                               "Game Rules:",
                                               "±Rule: Do not quote any of the Bots.",
                                               "±Rule: Do not quit the game before you are dead.",
                                               "±Rule: Do not vote yourself / get yourself killed on purpose",
                                               "±Rule: Do not talk once you're dead or voted off. ",
                                               "±Rule: Do not use a hard to type name.",
                                               "±Rule: Do not group together to ruin the game",
                                               "±Rule: DO NOT REVEAL YOUR PARTNER IF YOU ARE MAFIA",
                                               "",
                                               "±Game: Disobey them and you will be banned from mafia/muted according to the mod/admin's wishes!",
                                               ""];
                 dump(src,rules);
             };
             this.showThemes=function(src){
                 var l=[];
                 for(var t in mafia.themeManager.themes){
                     l.push(mafia.themeManager.themes[t].name);
                 }
                 var text="Installed themes are: "+l.join(", ");
                 sys.sendMessage(src,"+MafiaBot: "+text,mafiachan);
             };
             this.showThemeInfo=function(src){
                 mafia.themeManager.themeInfo.sort(function(a,b){
                                                       return a[0].localeCompare(b[0]);
                                                   });
                 var mess=[];
                 mess.push("<table><tr><th>Theme</th><th>URL</th><th>Author</th><th>Enabled</th></tr>");
                 for(var i=0;i<mafia.themeManager.themeInfo.length;++i){
                     var info=mafia.themeManager.themeInfo[i];
                     var theme=mafia.themeManager.themes[info[0].toLowerCase()];
                     if(!theme)
                         continue;
                     mess.push('<tr><td>'+theme.name+'</td><td><a href="'+info[1]+'">'+info[1]+'</a></td><td>'+(theme.author?theme.author:"unknown")+'</td><td>'+(theme.enabled?"yes":"no")+'</td></tr>');
                 }
                 mess.push("</table>");
                 sys.sendHtmlMessage(src,mess.join(""),mafiachan);
             }
             this.showPlayedGames=function(src){
                 var mess=[];
                 mess.push("<table><tr><th>Theme</th><th>Who started</th><th>When</th><th>Players</th></tr>");
                 var recentGames=PreviousGames.slice(-10);
                 var t=parseInt(sys.time());
                 for(var i=0;i<recentGames.length;++i){
                     var game=recentGames[i];
                     mess.push('<tr><td>'+game.what+'</td><td>'+game.who+'</td><td>'+getTimeString(game.when-t)+'</td><td>'+game.playerCount+'</td></tr>');
                 }
                 mess.push("</table>");
                 sys.sendHtmlMessage(src,mess.join(""),mafiachan);
             }
             this.isMafiaAdmin=function(src){
                 if(sys.auth(src)>=2)
                     return true;
                 if(['name123456790191298','etc8888888'].indexOf(sys.name(src).toLowerCase())>=0){
                     return true;
                 }
                 return false;
             }
             this.pushUser=function(src,name){
                 var id=sys.id(name);
                 if(id){
                     name=sys.name(id);
                     mafia.signups.push(name);
                     mafia.ips.push(sys.ip(id));
                 }
                 else{
                     mafia.signups.push(name);
                 }
                 sys.sendAll("±Game: "+name+" joined the game! (pushed by "+sys.name(src)+")",mafiachan);
             };
             this.slayUser=function(src,name){
                 name=mafia.correctCase(name);
                 if(mafia.isInGame(name)){
                     var player=mafia.players[name];
                     sys.sendAll("±Kill: "+player.name+" ("+player.role.translation+") was slayed by "+sys.name(src)+"!",mafiachan);
                     mafia.removePlayer(player);
                     mafia.testWin();
                 }
                 else{
                     sys.sendMessage(src,"+MafiaBot: No such target.",mafiachan);
                 }
             };
             this.addTheme=function(src,url){
                 if(!mafia.isMafiaAdmin(src)){
                     sys.sendMessage(src,"+MafiaBot: You need to be a mafia admin!",mafiachan);
                     return;
                 }
                 mafia.themeManager.loadWebTheme(url,true,false);
             }
             this.updateTheme=function(src,url){
                 if(!mafia.isMafiaAdmin(src)){
                     sys.sendMessage(src,"+MafiaBot: You need to be a mafia admin!",mafiachan);
                     return;
                 }
                 mafia.themeManager.loadWebTheme(url,true,true);
             };
             this.removeTheme=function(src,name){
                 if(!mafia.isMafiaAdmin(src)){
                     sys.sendMessage(src,"+MafiaBot: You need to be a mafia admin!",mafiachan);
                     return;
                 }
                 mafia.themeManager.remove(src,name);
             };
             this.disableTheme=function(src,name){
                 mafia.themeManager.disable(src,name);
             };
             this.enableTheme=function(src,name){
                 mafia.themeManager.enable(src,name);
             };
             this.importOld=function(src,name){ (function(){
                                                     sys.sendAll("+MafiaBot: Imported themes.",mafiachan)
                                                     this.themeManager.saveToFile(defaultTheme);
                                                     this.themeManager.saveToFile(insanityTheme);
                                                     this.themeManager.saveToFile(hpTheme);
                                                     this.themeManager.saveToFile(ssbbTheme);
                                                     this.themeManager.saveToFile(ffTheme);
                                                     this.themeManager.loadTheme(defaultTheme);
                                                     this.themeManager.loadTheme(insanityTheme);
                                                     this.themeManager.loadTheme(ffTheme);
                                                     this.themeManager.loadTheme(ssbbTheme);
                                                     this.themeManager.loadTheme(hpTheme);
                                                     this.themeManager.loadThemes();}).apply(mafia,[]);
                                              }
             this.commands={
                 user:{
                     commands:[this.showCommands,"To see the various commands."],
                     start:[this.startGame,"Starts a Game of Mafia."],
                     help:[this.showHelp,"For info on how to win in a game."],
                     roles:[this.showRoles,"For info on all the Roles in the game."],
                     rules:[this.showRules,"To see the Rules for the Game/Server."],
                     themes:[this.showThemes,"To view installed themes."],
                     themeinfo:[this.showThemeInfo,"To view installed themes (more details)."],
                     playedgames:[this.showPlayedGames,"To view recently played games"]
                 },
                 auth:{
                     push:[this.pushUser,"To push users to a Mafia game."],
                     slay:[this.slayUser,"To slay users in a Mafia game."],
                     end:[this.endGame,"To cancel a Mafia game!"],
                     add:[this.addTheme,"To add a Mafia Theme!"],
                     update:[this.updateTheme,"To update a Mafia Theme!"],
                     remove:[this.removeTheme,"To remove a Mafia Theme!"],
                     disable:[this.disableTheme,"To disable a Mafia Theme!"],
                     enable:[this.enableTheme,"To enable a disabled Mafia Theme!"],
                     importold:[this.importOld,"To import old themes."]
                 }
             };
             this.handleCommand=function(src,message){
                 var command;
                 var commandData='*';
                 var pos=message.indexOf(' ');
                 if(pos!=-1){
                     command=message.substring(0,pos).toLowerCase();
                     commandData=message.substr(pos+1);
                 }
                 else{
                     command=message.substr(0).toLowerCase();
                 }
                 if(command in this.commands["user"]){
                     this.commands["user"][command][0](src,commandData);
                     return;
                 }
                 if(this.state=="entry"){
                     if(command=="join"){
                         if(this.isInGame(sys.name(src))){
                             sys.sendMessage(src,"±Game: You already joined!",mafiachan);
                             return;
                         }
                         if(this.signups.length>=this.theme["roles"+this.theme.roleLists].length){
                             sys.sendMessage(src,"±Game: There can't be more than "+this.theme["roles"+this.theme.roleLists].length+" players!",mafiachan);
                             return;
                         }
                         var name=sys.name(src);
                         for(x in name){
                             var code=name.charCodeAt(x);
                             if(name[x]!=' '&&name[x]!='.'&&(code<'a'.charCodeAt(0)||code>'z'.charCodeAt(0))&&(code<'A'.charCodeAt(0)||code>'Z'.charCodeAt(0))&&name[x]!='-'&&name[x]!='_'&&name[x]!='<'&&name[x]!='>'&&(code<'0'.charCodeAt(0)||code>'9'.charCodeAt(0))){
                                 sys.sendMessage(src,"±Name: You're not allowed to have the following character in your name: "+name[x]+".",mafiachan);
                                 sys.sendMessage(src,"±Rule: You must change it if you want to join!",mafiachan);
                                 return;
                             }
                         }
                         this.signups.push(name);
                         this.ips.push(sys.ip(src));
                         sys.sendAll("±Game: "+name+" joined the game!",mafiachan);
                         if(this.signups.length==this.theme["roles"+this.theme.roleLists].length){
                             this.ticks=1;
                         }
                         return;
                     }
                     if(command=="leave"){
                         if(this.isInGame(sys.name(src))){
                             var name=sys.name(src);
                             delete this.ips[this.ips.indexOf(sys.ip(src))];
                             this.signups.splice(this.signups.indexOf(name),1);
                             sys.sendAll("±Game: "+name+" left the game!",mafiachan);
                             return;
                         }
                         else{
                             sys.sendMessage(src,"±Game: You haven't even joined!",mafiachan);
                             return;
                         }
                     }
                 }
                 else if(this.state=="night"){
                     var name=sys.name(src);
                     if(this.isInGame(name)&&this.hasCommand(name,command,"night")){
                         commandData=this.correctCase(commandData);
                         if(!this.isInGame(commandData)){
                             sys.sendMessage(src,"±Hint: That person is not playing!",mafiachan);
                             return;
                         }
                         var player=mafia.players[name];
                         var target=mafia.players[commandData];
                         if(player.role.actions.night[command].target!="Self"&&commandData==name){
                             sys.sendMessage(src,"±Hint: Nope, this won't work... You can't target yourself!",mafiachan);
                             return;
                         }
                         else if(player.role.actions.night[command].target=='AnyButTeam'&&player.role.side==target.role.side||player.role.actions.night[command].target=='AnyButRole'&&player.role.role==target.role.role){
                             sys.sendMessage(src,"±Hint: Nope, this won't work... You can't target your partners!",mafiachan);
                             return;
                         }
                         sys.sendMessage(src,"±Game: You have chosen to "+command+" "+commandData+"!",mafiachan);
                         this.setTarget(player,target,command);
                         var broadcast=player.role.actions.night[command].broadcast;
                         if(broadcast!==undefined){
                             var team=[];
                             if(broadcast=="team"){
                                 team=this.getPlayersForTeam(player.role.side);
                             }
                             else if(broadcast=="role"){
                                 team=this.getPlayersForRole(player.role.role);
                             }
                             for(x in team){
                                 if(team[x]!=name){
                                     this.sendPlayer(team[x],"±Game: Your partner(s) have decided to "+command+" '"+commandData+"'!");
                                 }
                             }
                         }
                         if("avoidHax" in player.role.actions&&player.role.actions.avoidHax.indexOf(command)!=-1){
                             return;
                         }
                         var haxRoles=mafia.theme.getHaxRolesFor(command);
                         for(var i in haxRoles){
                             var role=haxRoles[i];
                             var haxPlayers=this.getPlayersForRole(role);
                             for(j in haxPlayers){
                                 var haxPlayer=haxPlayers[j];
                                 var r=Math.random();
                                 var roleName=this.theme.trside(player.role.side);
                                 if(r<mafia.theme.roles[role].actions.hax[command].revealTeam){
                                     this.sendPlayer(haxPlayer,"±Game: The "+roleName+" are going to kill "+commandData +"!");
                                 }
                                 if(r<mafia.theme.roles[role].actions.hax[command].revealPlayer){
                                     this.sendPlayer(haxPlayer,"±Game: "+name+" is one of the "+roleName+"!");
                                 }
                             }
                         }
                         return;
                     }
                 }
                 else if(this.state=="day"){
                     if(this.isInGame(sys.name(src))&&command=="vote"){
                         commandData=this.correctCase(commandData);
                         if(!this.isInGame(commandData)){
                             sys.sendMessage(src,"±Game: That person is not playing!",mafiachan);
                             return;
                         }
                         if(sys.name(src)in this.votes){
                             sys.sendMessage(src,"±Rule: You already voted!",mafiachan);
                             return;
                         }
                         sys.sendAll("±Game: "+sys.name(src)+" voted for "+commandData+"!",mafiachan);
                         this.votes[sys.name(src)]=commandData;
                         this.voteCount+=1;
                         if(this.voteCount==Object.keys(mafia.players).length){
                             mafia.ticks=1;
                         }
                         else if(mafia.ticks<8){
                             mafia.ticks=8;
                         }
                         return;
                     }
                 }
                 else if(mafia.state=="standby"){
                     var name=sys.name(src);
                     if(this.isInGame(name)&&this.hasCommand(name,command,"standby")){
                         if(mafia.players[name].role.actions.standby[command].hasOwnProperty("command"))
                             command=mafia.players[name].role.actions.standby[command].command
                         if(command=="kill"){
                             if(mafia.players[name].dayKill){
                                 sys.sendMessage(src,"±Game: You already killed!",mafiachan);
                                 return;
                             }
                             commandData=this.correctCase(commandData); 
                             if(!this.isInGame(commandData)){
                                 sys.sendMessage(src,"±Game: That person is not playing!",mafiachan);
                                 return;
                             }
                             sys.sendAll("*** ************************************************************************************",mafiachan);
                             sys.sendAll("±Game: "+this.players[name].role.actions.standby[command].killmsg.replace(/~Self~/g,name).replace(/~Target~/g,commandData),mafiachan);
                             this.players[name].dayKill=true;
                             this.kill(mafia.players[commandData]);
                             if(this.testWin()){
                                 return;
                             }
                             sys.sendAll("*** ************************************************************************************",mafiachan);}
                         return;
                     }
                 }
                 if(command=="join"){
                     sys.sendMessage(src,"±Game: You can't join now!",mafiachan);
                     return;
                 }
                 if(!mafia.isMafiaAdmin(src))
                     throw("no valid command");
                 if(command in this.commands["auth"]){
                     this.commands["auth"][command][0](src,commandData);
                     return;
                 }
                 throw("no valid command");
             }
         }();
     }
     ,
     initVal : function(key,val) { // Astruvis' Edits - Required edit to variable structure
	 if (typeof server[key] == 'undefined') {
	     server[key] = val; }
	 return; }
     // End edits
     ,
     init : function() {
         lastMemUpdate = 0;
	 this.mafiaLoad();
         
         var dwlist = ["Rattata", "Raticate", "Nidoran-F", "Nidorina", "Nidoqueen", "Nidoran-M", "Nidorino", "Nidoking", "Oddish", "Gloom", "Vileplume", "Bellossom", "Bellsprout", "Weepinbell", "Victreebel", "Ponyta", "Rapidash", "Farfetch'd", "Doduo", "Dodrio", "Exeggcute", "Exeggutor", "Lickitung", "Lickilicky", "Tangela", "Tangrowth", "Kangaskhan", "Sentret", "Furret", "Cleffa", "Clefairy", "Clefable", "Igglybuff", "Jigglypuff", "Wigglytuff", "Mareep", "Flaaffy", "Ampharos", "Hoppip", "Skiploom", "Jumpluff", "Sunkern", "Sunflora", "Stantler", "Poochyena", "Mightyena", "Lotad", "Ludicolo", "Lombre", "Taillow", "Swellow", "Surskit", "Masquerain", "Bidoof", "Bibarel", "Shinx", "Luxio", "Luxray", "Psyduck", "Golduck", "Growlithe", "Arcanine", "Scyther", "Scizor", "Tauros", "Azurill", "Marill", "Azumarill", "Bonsly", "Sudowoodo", "Girafarig", "Miltank", "Zigzagoon", "Linoone", "Electrike", "Manectric", "Castform", "Pachirisu", "Buneary", "Lopunny", "Glameow", "Purugly", "Natu", "Xatu", "Skitty", "Delcatty", "Eevee", "Vaporeon", "Jolteon", "Flareon", "Espeon", "Umbreon", "Leafeon", "Glaceon", "Eevee", "Bulbasaur", "Charmander", "Squirtle", "Ivysaur", "Venusaur", "Charmeleon", "Charizard", "Wartortle", "Blastoise", "Croagunk", "Toxicroak", "Turtwig", "Grotle", "Torterra", "Chimchar", "Infernape", "Monferno", "Piplup", "Prinplup", "Empoleon", "Treecko", "Sceptile", "Grovyle", "Torchic", "Combusken", "Blaziken", "Mudkip", "Marshtomp", "Swampert", "Caterpie", "Metapod", "Butterfree", "Pidgey", "Pidgeotto", "Pidgeot", "Spearow", "Fearow", "Zubat", "Golbat", "Crobat", "Aerodactyl", "Hoothoot", "Noctowl", "Ledyba", "Ledian", "Yanma", "Yanmega", "Murkrow", "Honchkrow", "Delibird", "Wingull", "Pelipper", "Swablu", "Altaria", "Starly", "Staravia", "Staraptor", "Gligar", "Gliscor", "Drifloon", "Drifblim", "Skarmory", "Tropius", "Chatot", "Slowpoke", "Slowbro", "Slowking", "Krabby", "Kingler", "Horsea", "Seadra", "Kingdra", "Goldeen", "Seaking", "Magikarp", "Gyarados", "Omanyte", "Omastar", "Kabuto", "Kabutops", "Wooper", "Quagsire", "Qwilfish", "Corsola", "Remoraid", "Octillery", "Mantine", "Mantyke", "Carvanha", "Sharpedo", "Wailmer", "Wailord", "Barboach", "Whiscash", "Clamperl", "Gorebyss", "Huntail", "Relicanth", "Luvdisc", "Buizel", "Floatzel", "Finneon", "Lumineon", "Tentacool", "Tentacruel", "Corphish", "Crawdaunt", "Lileep", "Cradily", "Anorith", "Armaldo", "Feebas", "Milotic", "Shellos", "Gastrodon", "Lapras", "Dratini", "Dragonair", "Dragonite", "Elekid", "Electabuzz", "Electivire", "Poliwag", "Poliwrath", "Politoed", "Poliwhirl", "Vulpix", "Ninetales", "Musharna", "Munna", "Darmanitan", "Darumaka", "Mamoswine", "Togekiss", "Burmy", "Wormadam", "Mothim"];
         dwpokemons = [];
         for(var dwpok in dwlist) {
             dwpokemons.push(sys.pokeNum(dwlist[dwpok]));
         }
         
         
         var lclist = ["Bulbasaur", "Charmander", "Squirtle", "Croagunk", "Turtwig", "Chimchar", "Piplup", "Treecko","Torchic","Mudkip"]
         lcpokemons = [];
         for(var dwpok in lclist) {
             lcpokemons.push(sys.pokeNum(lclist[dwpok]));
         }
         
         var inconsistentList = ["Remoraid", "Bidoof", "Snorunt", "Smeargle", "Bibarel", "Octillery", "Glalie"];
         inpokemons = [];
         for(var inpok in inconsistentList) {
             inpokemons.push(sys.pokeNum(inconsistentList[inpok]));
         }
         
         var breedingList = ["Eevee", "Umbreon", "Espeon", "Vaporeon", "Jolteon", "Flareon", "Leafeon", "Glaceon", "Bulbasaur", "Ivysaur", "Venusaur", "Charmander", "Charmeleon", "Charizard", "Squirtle", "Wartortle", "Blastoise", "Croagunk", "Toxicroak", "Turtwig", "Grotle", "Torterra", "Chimchar", "Monferno", "Infernape", "Piplup", "Prinplup", "Empoleon", "Treecko", "Grovyle", "Sceptile", "Torchic", "Combusken", "Blaziken", "Mudkip", "Marshtomp", "Swampert"];
         breedingpokemons = [];
         for(var inpok in breedingList) {
             breedingpokemons.push(sys.pokeNum(breedingList[inpok]));
         }

	 if(typeof(MOTD)=='undefined'){
	     var MOTD = sys.getVal("MOTD")
	 }

         key = function(a,b) {
             return a + "*" + sys.name(b);
         }

         semiUbers = [];
         
         var tempU = new Array(150,249,250,382,383,384,483,484,487,505);
         for (x in tempU) {
             semiUbers[tempU[x]] = true;
         }
         
         saveKey = function(thing, id, val) {
             sys.saveVal(key(thing,id), val);
         }
         
         getKey = function(thing, id) {
             return sys.getVal(key(thing,id));
         }
         
         hasBan = function(id, poke) {
             return clauses[id].indexOf("*" + poke + "*") != -1;
         }
         
         cmp = function(a, b) {
             return a.toLowerCase() == b.toLowerCase();
         }
         
         if (typeof(varsCreated) != 'undefined')
             return;

	 // Astruvis' Edits - Variable structure
	 if (typeof server == 'undefined') {
	     server = []; }
	 script.initVal('pollmode',0);
	 script.initVal('pollstr','');
	 script.initVal('pollvotes',[]);
	 // End edit block

         battlesStopped = false;
         channelUsers = [];
         channelTopics = [];

         MOTD = [];
         
         guestbook = sys.getVal("guestbook");

         pingstatus = [];

         muteall = false;
         
         muted = [];
         maxPlayersOnline = 0;
         
         lineCount = 0;
         if (typeof tourmode == 'undefined') {
	     tourmode = 0; }
         border = "»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»»:";

	 getNature = function(id, slot) {
             natureMsg = new Array("Hardy*","Docile*","Serious*","Bashful*","Quirky*","Lonely*(+Atk, -Def)","Brave*(+Atk, -Spd)","Adamant*(+Atk, -SAtk)","Naughty*(+Atk, -SDef)","Bold*(-Atk, +Def)","Relaxed*(-Spd, +Def)","Impish*(-SAtk, +Def)","Lax*(-SDef, +Def)","Timid*(-Atk, +Spd)","Hasty*(-Def, +Spd)","Jolly*(-SAtk, +Spd)","Naive*(-SDef, +Spd)","Modest*(-Atk, +SAtk)","Mild*(-Def, +SAtk)","Quiet*(-Spd, +SAtk)","Rash*(-SDef, +SAtk)","Calm*(-Atk, +SDef)","Gentle*(-Def, +SDef)","Sassy*(-Spd, +SDef)","Careful*(-SAtk, +SDef)");
             for (t in natureMsg) {
                 var nMsg = natureMsg[t].split("*");
                 for (var team = 0; team < sys.teamCount(id); team++) {
                     for (var i = 0; i < 6; ++i) {
                         if (nMsg[0] == sys.nature(sys.teamPokeNature(id, team, slot))) {
                             return    nMsg[0] + " Nature " + nMsg[1];
	                 }
        	     }
	         }  
             }
         }

         rules = [ "",
                   "***Da Rulez***",
                   "",
                   "Rule #1 - Do Not Abuse UPPERCASE LETTERS:",
                   "- The occasional word in UPPERCASE LETTERS is acceptable, however repeated use is not.",
                   "Rule #2 - No Flooding the Chat:",
                   "- Please do not post a large amount of short messages when you can easily post one or two long messages.",
                   "Rule #3 - Do not Challenge Spam:",
                   "- If a player refuses your challenge, this means they do not want to battle you. Find someone else to battle with.",
                   "Rule #4 - No Advertising:",
                   "- There will be absolutely no advertising on the server.",
                   "Rule #5 - No Obscene or Pornographic Content Allowed:",
                   "- This includes links, images, and any other kind of media. This will result in an instant ban.",
                   "Rule #6 - Do not ask for Auth:",
                                         "- Authority is given upon merit. By asking you have pretty much eliminated your chances at becoming a member of authority in the future.",
                                         "Rule #7 - Do not Insult Auth:",
                                         "- Insulting members of authority will result in immediate punishment.",
                                         "Rule #8 - No minimodding:",
                                         "- The server has moderators for a reason. If someone breaks the rules, alert the authorities. Do not try to moderate the situation yourself.",
                                         "Rule #9 - No asking for personal info:",
                                         "- You cannot ask anyone where they live, etc.. .",
                                         "Rule #10 - No torrents allowed:",
                                         "- Torrents are illegal, and it can get you into big legal trouble. If you see a link to a torrent, DO NOT CLICK ON IT. Breaking said rule results in a ban.",
                                         "Rule #11 - The server owners make the authority level changes:",
                                         "- From now on, the server owners make the authority level changes. All other people without permission will suffer consequences after one to three warnings.",
                                         "Rule #12 - NEVER, EVER, make an authority member upset:",
                                         "- They have the ability to punish you, depending on the severity. This is there for a reason.",
                                         ""    ];

         pokeNatures = [];
         
         var list = "Heatran-Eruption/Quiet=Suicune-ExtremeSpeed/Relaxed|Sheer Cold/Relaxed|Aqua Ring/Relaxed|Air Slash/Relaxed=Raikou-ExtremeSpeed/Rash|Weather Ball/Rash|Zap Cannon/Rash|Aura Sphere/Rash=Entei-ExtremeSpeed/Adamant|Flare Blitz/Adamant|Howl/Adamant|Crush Claw/Adamant";
         
         var sepPokes = list.split('=');
         for (var x in sepPokes) {
             sepMovesPoke = sepPokes[x].split('-');
             sepMoves = sepMovesPoke[1].split('|');
             
             var poke = sys.pokeNum(sepMovesPoke[0]);
             pokeNatures[poke] = [];
             
             for (y in sepMoves) {
                 movenat = sepMoves[y].split('/');
                 pokeNatures[poke][sys.moveNum(movenat[0])] = sys.natureNum(movenat[1]);
             }
         }
     }

     ,

     importable : function(id, compactible) {
         /*
          Tyranitar (M) @ Choice Scarf
          Lvl: 100
          Trait: Sand Stream
          IVs: 0 Spd
          EVs: 4 HP / 252 Atk / 252 Spd
          Jolly Nature (+Spd, -SAtk)
          - Stone Edge
          - Crunch
          - Superpower
          - Pursuit
          */
         if (compactible === undefined) compactible = false;
         var nature_effects = {"Adamant": "(+Atk, -SAtk)", "Bold": "(+Def, -Atk)"}
         var genders = {0: ' (N)', 1: ' (M)', 2: ' (F)'};
         var stat = {0: 'HP', 1: 'Atk', 2: 'Def', 3: 'SAtk', 4: 'SDef', 5:'Spd'};
         var hpnum = sys.moveNum("Hidden Power");
         var ret = [];
         for (var team = 0; team < sys.teamCount(id); team++) {
             for (var i = 0; i < 6; ++i) {
                 var poke = sys.teamPoke(id, team, i);
                 if (poke === undefined)
                     continue;

                 var item = sys.teamPokeItem(id, team, i);
                 item = item !== undefined ? sys.item(item) : "(no item)";
                 ret.push(sys.pokemon(poke) + genders[sys.teamPokeGender(id, team, i)] + " @ " + item );
                 ret.push('Ability: ' + sys.ability(sys.teamPokeAbility(id, team, i)));
                 var level = sys.teamPokeLevel(id, team, i);
                 if (!compactible && level != 100) ret.push('Lvl: ' + level);

                 var ivs = [];
                 var evs = [];
                 var hpinfo = [sys.gen(id,team)];
                 for (var j = 0; j < 6; ++j) {
                     var iv = sys.teamPokeDV(id, team, i, j);
                     if (iv != 31) ivs.push(iv + " " + stat[j]);
                     var ev = sys.teamPokeEV(id, team, i, j);
                     if (ev != 0) evs.push(ev + " " + stat[j]);
                     hpinfo.push(iv);
                 }
                 if (!compactible && ivs.length > 0)
                     ret.push('IVs: ' + ivs.join(" / "));
                 if (evs.length > 0)
                     ret.push('EVs: ' + evs.join(" / "));

                 ret.push(sys.nature(sys.teamPokeNature(id, team, i)) + " Nature"); // + (+Spd, -Atk)

                 for (var j = 0; j < 4; ++j) {
                     var move = sys.teamPokeMove(id, team, i, j);
                     if (move !== undefined) {
                         ret.push('- ' + sys.move(move) + (move == hpnum ? ' [' + sys.type(sys.hiddenPowerType.apply(sys, hpinfo)) + ']':''));
                     }
                 }
                 ret.push("")
             }
             return ret;
         }
     }
     ,

     afterChannelCreated : function (chan, name, src) {
         if (src == 0)
             return;
         
         channelUsers[chan] = src;
     }

     ,

     afterChannelJoin : function(player, chan) {
         if (typeof(channelTopics[chan]) != 'undefined') {
             sys.sendHtmlMessage(player, '<font color="blue"><timestamp/> <b>Welcome Message:</b></font> ' + channelTopics[chan], chan);
         }
         if (typeof(channelUsers[chan]) != 'undefined' && player == channelUsers[chan]) {
             sys.sendHtmlMessage(player, '<font color="#3DAA68"><timestamp/> <b>+TopicBot:</b></font> use /topic [topic] to change the welcome message of this channel.', chan);
             return;
         }
     }

     ,

     beforeChannelDestroyed : function(channel) {
         if (channel == "The Treehouse") {
             sys.stopEvent();
             return;
         }
         if (channel == "Mafia Channel") {
             sys.createChannel("Mafia Channel");
             return;
         }
         if (channel == "Sendall Watch") {
             sys.createChannel("Sendall Watch");
             return;
         }
         
         delete channelUsers[channel]
         delete channelTopics[channel]
     }
     ,

     afterNewMessage : function (message) {
         if (message == "Script Check: OK") {
             /*            var c;
              for (c=0;c<2999;c++) {
              sys.sendAll("");
              }
              sys.clearChat();
              sys.sendHtmlAll('<font color="blue"><timestamp/> *** The scripts have been updated! Hooray! ***</font>');*/
             if (typeof(scriptChecks)=='undefined')
                 scriptChecks = 0;
             scriptChecks += 1;
             this.init();
         }
         if (message == "Announcement changed.") {
             sys.sendHtmlAll('<timestamp/> The announcement has been changed.');
	 }
         if (message == "The description of the server changed.") {
             sys.sendHtmlAll('<timestamp/> The server description has been changed.');
	 }
         if (message == "The server is now private.") {
             //sys.sendHtmlAll('<timestamp/> The server is now in private mode.');
	 }
         if (message == "The server is now public.") {
             //sys.sendHtmlAll('<timestamp/> The server is now in public mode.');
	 }
         if  (message == "Error when connecting to the registry. Will restart in 30 seconds") {
             //sys.sendHtmlAll('<timestamp/> The server failed to connect to the registry for some reason.');
             //sys.makeServerPublic(false);
             //sys.makeServerPublic(true);
         }
	 if  (message == "The registry acknowledged the server.") {
             //sys.sendHtmlAll('<timestamp/> The server should now be on the registry.');
	 }
	 if  (message == "The name of the server is already in use. Please change it in Options -> Config.") {
             //sys.sendHtmlAll('<timestamp/> The server was unable to connect due to the server being on the registry. Now it has to wait until the server gets off the registry.');
             //sys.makeServerPublic(false);
             //sys.makeServerPublic(true);
	 }
     }

     ,

     afterLogIn : function(src) {

         // Send a message!

         if ( sys.ip(src).substr(0, 12) == "66.74.54.101") {
             //sys.sendAll("Gray: HEY! GET YOUR ASS OUT HERE!.");
             return;
         }

         // Commence kick

         if ( sys.ip(src).substr(0, 7) == "84.212.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am a God.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "98.204.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am a God.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "66.131.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "63.19.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "98.253.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "64.134.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 7) == "71.167.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 6) == "66.87.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 7) == "173.74.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "209.99.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "98.244.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "92.41.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 6) == "98.22.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "65.10.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "81.94.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "50.28.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "123.49.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "173.63.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "69.204.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "74.177.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "69.171.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "74.13.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "69.158.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "93.65.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 9) == "75.65.16.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "70.180.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "75.190.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "184.5.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "98.249.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "174.252.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "174.141.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "174.36.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "173.2.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "76.101.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "67.232.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "70.189.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "216.169.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "108.208.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 9) == "76.20.54.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "74.13.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "24.125.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 9) == "68.11.43.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 10) == "184.144.10") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "184.149.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "173.217.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "67.211.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "98.217.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 6) == "78.32.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "76.19.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "90.216.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 6) == "68.41.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "69.120.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "98.190.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "188.28.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "115.64.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "74.115.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             //sys.ban(src);
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "213.89.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "66.199.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "116.71.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 10) == "65.95.183.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "128.187.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "118.227.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "67.221.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 8) == "142.167.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "68.73.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "76.21.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "94.46.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "74.82.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "84.19.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "70.33.96") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "199.255.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 11) == "67.243.150.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "71.224.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "69.118.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "204.210.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "184.164.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "71.180.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "68.187.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "96.240.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "72.79.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "72.20.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "88.109.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "74.108.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "88.77.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 6) == "66.87.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.ip(src).substr(0, 8) == "190.135.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 8) == "204.236.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "41.96.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "74.64.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "69.125.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "70.29.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "70.24.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "24.161.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         //if ( sys.ip(src).substr(0, 7) == "24.247.") {
         //sys.kick(src);
         //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
         //return;
         //}

         if ( sys.auth(src) < 1 && sys.ip(src).substr(0, 7) == "67.172.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "98.238.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "156.34.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "67.243.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 5) == "75.1.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "76.182.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "67.228.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 10) == "74.73.173.") {
             sys.kick(src);
             //sys.sendAll("+BossBot: Target destroyed, for I am the boss. BOSS BOSS BOSS.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "109.35.") {
             //sys.sendMessage(src, "+Bot: We do not tolerate your kind here.");
             sys.kick(src);
             //sys.sendAll("+Bot: Eliminated our target.");
             return;
         }

         if ( sys.ip(src).substr(0, 7) == "50.101.") {
             //sys.sendMessage(src, "+Bot: We do not tolerate your kind here.");
             sys.kick(src);
             //sys.sendAll("+Bot: Eliminated our target.");
             return;
         }

         if ( sys.ip(src).substr(0, 6) == "67.85.") {
             //sys.sendMessage(src, "+Bot: We do not tolerate your kind here.");
             sys.kick(src);
             //sys.sendAll("+Bot: Eliminated our target.");
             return;
         }

         var name = sys.name(src).toLowerCase();
         if (name.indexOf("doj") != -1 || name.indexOf("d.o.j") != -1 || name.indexOf("odj") != -1 || name.indexOf("hitler") != -1 || name.indexOf("skram") != -1 || name.indexOf("smogon") != -1 || name.indexOf("supa") != -1 || name.indexOf("fb") != -1 || name.indexOf("skyrpz") != -1 || name.indexOf("gfm2") != -1 || name.indexOf("battle") != -1 || name.indexOf("69") != -1 || name.indexOf("trl") != -1 || name.indexOf("smgn") != -1 || name.indexOf("smegen") != -1 || name.indexOf("trell") != -1 || name.indexOf("smawgawn") != -1 || name.indexOf("fag") != -1 || name.indexOf("update") != -1){ // this checks for presence of t3h tr0ll
             sys.sendAll("+ArceusBot: You shall receive JUDGMENT!");
             sys.setTimer("sys.kick(" + src + ")", 1, 0);
             return;
         }

         if (sys.name(src) == "Axel" || sys.name(src) == "Saeyru" || sys.name(src) == "Misora") {
             sys.changeAway(src, true);
         }

         if (name.indexOf("PokemonTrainer") != -1){ // this checks for presence of t3h tr0ll
             sys.sendAll("+ArceusBot: You shall receive JUDGMENT!");
             sys.setTimer("sys.kick(" + src + ")", 1, 0);
             return;
         }

         if (name.indexOf("Timmy") != -1){ // this checks for presence of t3h tr0ll
             sys.sendAll("+ArceusBot: You shall receive JUDGMENT!");
             sys.setTimer("sys.kick(" + src + ")", 1, 0);
             return;
         }

         if (sys.name(src).indexOf("PO") != -1 || sys.name(src).indexOf("UNC") != -1 || sys.name(src).indexOf("PL") != -1 || sys.name(src).indexOf("UL") != -1 || sys.name(src).indexOf("GO") != -1){ // this checks for presence of t3h tr0ll
             sys.sendAll("+ArceusBot: You shall receive JUDGMENT!");
             sys.setTimer("sys.kick(" + src + ")", 1, 0);
             return;
         }

         sys.sendMessage(src, "*** Type in /Rules to see the rules. ***");
         sys.sendMessage(src, "+InstructionBot: To get started, use /commands to bring up the command categories. You can use /updates [page] at any time to view the current updates, where [page] is a number.");

         if (sys.getVal("muted_*" + sys.ip(src)) == "true") 
             muted[src] = true;
         else
             muted[src] = false;

         
         if (sys.numPlayers() > maxPlayersOnline) {
             maxPlayersOnline = sys.numPlayers();
         }

         if (maxPlayersOnline > sys.getVal("MaxPlayersOnline")) {
             sys.saveVal("MaxPlayersOnline", maxPlayersOnline);
         }

         sys.sendMessage(src, "+CountBot: Number of players online is " + sys.numPlayers() + ". High score " + sys.getVal("MaxPlayersOnline") + ".");
         sys.sendMessage(src, "");
         if (tourmode == 1 && prize == undefined){
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
             sys.sendMessage(src,"*** A tournament (" + tourtier + ") is in its signup phase. Spots left: " + this.tourSpots() + ". The prize is nothing. ***");
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");

         } else if (tourmode == 1){
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
             sys.sendMessage(src,"*** A tournament (" + tourtier + ") is in its signup phase. Spots left: " + this.tourSpots() + ". The prize is " + prize + " ***");
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");

         } else if (tourmode == 2 && prize == undefined){
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
             sys.sendMessage(src, "+Bot: A tournament (" + tourtier + ") is currently running. The prize is nothing.");
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
         } else if (tourmode == 2){
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
             sys.sendMessage(src, "+Bot: A tournament (" + tourtier + ") is currently running. The prize is " + prize);
             sys.sendMessage(src, "");
             sys.sendMessage(src, border);
             sys.sendMessage(src, "");
         }

         sys.sendMessage(src, "");

         pingstatus[src] = getKey("pingstatus", src) == "true";
         
         sys.sendHtmlMessage(src, "<timestamp/> <b>Message of the Day:</b> " + MOTD);
         sys.sendMessage(src, "");

	 greetmessage = sys.rand(1, 10)
	 if (greetmessage == 1){
             sys.sendMessage(src, "+WelcomeBot: Greetings, " + sys.name(src) + ".");
	 } else if (greetmessage == 2){
             sys.sendMessage(src, "+WelcomeBot: Hello, " + sys.name(src) + ".");
	 } else if (greetmessage == 3){
             sys.sendMessage(src, "+WelcomeBot: Welcome, " + sys.name(src) + ".");
	 } else if (greetmessage == 4){
             sys.sendMessage(src, "+WelcomeBot: It is nice to see you, " + sys.name(src) + ".");
	 } else if (greetmessage == 5){
             sys.sendMessage(src, "+WelcomeBot: It is good to see you, " + sys.name(src) + ".");
	 } else if (greetmessage == 6){
             sys.sendMessage(src, "+WelcomeBot: It is great to see you, " + sys.name(src) + ".");
	 } else if (greetmessage == 7){
             sys.sendMessage(src, "+WelcomeBot: Enjoy your stay, " + sys.name(src) + ".");
	 } else if (greetmessage == 8){
             sys.sendMessage(src, "+WelcomeBot: Have a blast, " + sys.name(src) + ".");
	 } else {
             sys.sendMessage(src, "+WelcomeBot: Kick back, relax, and have fun, " + sys.name(src) + ".");
         }

         if (muteall) {
             sys.sendMessage(src, "");
             sys.sendMessage(src, "+MuteBot: Massmute is currently in effect.");
             sys.sendMessage(src, "");
         } else {
             sys.sendMessage(src, "");
             sys.sendMessage(src, "+MuteBot: Massmute is currently not in effect.");
             sys.sendMessage(src, "");
         }

         if (battlesStopped) {
             sys.sendMessage(src, "+BattleBot: Battles are disabled here because for whatever reason, they crash the server. So this server is mainly for RPs and Mafia.");
             sys.sendMessage(src, "");
	     return;
         } else {
             //                sys.sendMessage(src, "+BattleBot: Battles are enabled when they should not be to eliminate, or at least reduce, the chance of crashing the server for some bizarre reason.");
             sys.sendMessage(src, "+BattleBot: Battles are enabled to check for crashes with battling in the new version.");
             sys.sendMessage(src, "");
	     return;
         }
     }
     ,
     afterChangeTeam : function(src)
     {
         for (var team = 0; team < sys.teamCount(src); team++) {
             if (sys.gen(src, team) >= 4) {
                 for (var i = 0; i < 6; i++) {
                     var poke = sys.teamPoke(src, team, i);
                     if (poke in pokeNatures) {
                         for (x in pokeNatures[poke]) {
                             if (sys.hasTeamPokeMove(src, team, i, x) && sys.teamPokeNature(src, team, i) != pokeNatures[poke][x])
                             {
                                 checkbot.sendMessage(src, "" + sys.pokemon(poke) + " with " + sys.move(x) + " must be of the " + sys.nature(pokeNatures[poke][x]) + " nature. Change it in the team builder.");
                                 sys.changePokeNum(src, team, i, 0);
                             }
                         }
                     }
                 }
             }
         }

         var name = sys.name(src).toLowerCase();
         if (name.indexOf("doj") != -1 || name.indexOf("imnew") != -1 || name.indexOf("odj") != -1 || name.indexOf("hitler") != -1 || name.indexOf("skram") != -1 || name.indexOf("skarm") != -1 || name.indexOf("smogon") != -1 || name.indexOf("supa") != -1 || name.indexOf("skyrpz") != -1 || name.indexOf("gfm2") != -1 || name.indexOf("battle") != -1 || name.indexOf("69") != -1 || name.indexOf("[inf]") != -1 || name.indexOf("trl") != -1 || name.indexOf("smgn") != -1 || name.indexOf("smegen") != -1 || name.indexOf("trell") != -1 || name.indexOf("smawgawn") != -1 || name.indexOf("fag") != -1 || name.indexOf("update") != -1){ // this checks for presence of t3h tr0ll
             sys.sendAll("+ArceusBot: You shall receive JUDGMENT!");
             sys.setTimer("sys.kick(" + src + ")", 1, 0);
             return;
         }

         this.dreamWorldAbilitiesCheck(src, false);
         this.littleCupCheck(src, false);
         this.inconsistentCheck(src, false);
         this.monotypecheck(src);
         this.weatherlesstiercheck(src);

     }
     ,
     step:function() {
         mafia.tickDown();
         return; 
     }
     ,
     beforeChatMessage: function(src, message, chan) {
         
         channel = chan;

         if (message.search(/[\u2028\u0821\u00AD\u0823\u0808\u0804\u0814\u0819\u0853\u0813\u0819\u0817\u0816\u0826\u0858\u0797\u0805\u0787\u0872\u0877\u0783\u0771\u2029\u200E\u200F\u202A\u202B\u202C\u202D\u202E\u8235\u8234\u8236\u8238\u1161\u3657\u0E49]/) != -1) {
             sys.stopEvent();
             sys.sendMessage(src, "+Bot: What are you trying to do?", chan);
             return;
         }

         var m = message.toLowerCase();
         if (m.indexOf("&shy;") != -1 || m.indexOf("disconnected from server") != -1 || m.indexOf("&#173;") != -1 || m.indexOf("&#xad;") != -1 || m.indexOf("bit.ly") != -1 || m.indexOf("smogonspace.tk") != -1 || m.indexOf("topgoodjerseys.com") != -1 || m.indexOf("2girls1finger.org") != -1 || m.indexOf("mudkipz.ws") != -1 || m.indexOf(".dk") != -1 || m.indexOf("fingerslam.com") != -1 || m.indexOf("1guy2needles.com") != -1 || m.indexOf("2girls1cup.nl") != -1 || m.indexOf("smogonscouting") != -1 || m.indexOf("omg.png") != -1 || m.indexOf("pocketmonsters") != -1 || m.indexOf("googlehammer.com") != -1 || m.indexOf("mylazysundays.com") != -1 || m.indexOf("tinyurl.com") != -1 || m.indexOf("spin.com") != -1 || m.indexOf(".dk") != -1  || m.indexOf("2girls1cup.ws") != -1 || m.indexOf("redtube.com") != -1 || m.indexOf("goo.gl") != -1 || m.indexOf("goatse.cx") != -1 || m.indexOf("lemonparty.org") != -1 || m.indexOf("stileproject.com") != -1 || m.indexOf("bagslap.com") != -1 || m.indexOf("mudfall.com") != -1 || m.indexOf("youaresogay.com") != -1 || m.indexOf("tubgirl.com") != -1 || m.indexOf("hai2u.com") != -1 || m.indexOf("bottleguy.com") != -1 || m.indexOf("lastmeasure.com") != -1 || m.indexOf("smoke.rotten.com") != -1 || m.indexOf("blacksnake.com") != -1 || m.indexOf("thepounder.com") != -1 || m.indexOf("zentastic.com") != -1 || m.indexOf("cyberscat.com") != -1 || m.indexOf("brad.com") != -1 || m.indexOf("dontwatch.us") != -1 || m.indexOf("ourspace.be") != -1 || m.indexOf("denial.be") != -1 || m.indexOf("fnord.org") != -1 || m.indexOf("detroithardcore.com") != -1 || m.indexOf("goatse.ca") != -1 || m.indexOf("mongface.com") != -1 || m.indexOf("populationpaste.com") != -1 || m.indexOf("phbreak.org") != -1 || m.indexOf("tubboy.net") != -1 || m.indexOf("supermodelsfart.com") != -1 || m.indexOf("fuck.org") != -1 || m.indexOf("jiztini.com") != -1 || m.indexOf("shitpassion.com") != -1 || m.indexOf("cadaver.org") != -1 || m.indexOf("exet.nu") != -1 || m.indexOf("iheartu.tk") != -1 || m.indexOf("goregasm.com") != -1 || m.indexOf("40inchplus.com") != -1 || m.indexOf("dr47.com") != -1 || m.indexOf("nimp.org") != -1 || m.indexOf("google.copm") != -1 || m.indexOf("goggle.com") != -1 || m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
             sys.stopEvent();
             sys.sendMessage(src, "+Bot: What are you trying to do?", chan);
             return;
         }

         if (sys.auth(src) < 4 && muted[src] === true && message != "!join" && message != "/rules" && message != "/join" && message != "!rules" && message != "/leave" && message != "!leave" && message != "!mutecheck" && message != "/mutecheck" && message != "!battlecheck" && message != "/battlecheck" && message != "!massmutecheck" && message != "/massmutecheck" && message != "/updates" && message != "!updates" && message != "/updates 1" && message != "!updates 1" && message != "/updates 2" && message != "!updates 2" && message != "/updates 3" && message != "!updates 3" && message != "/updates 4" && message != "!updates 4" && message != "/updates 5" && message != "!updates 5") { // Hack hack hack all day long, hack hack hack while I sing this song.
             sys.sendMessage(src, "+Bot: You are muted.");
             sys.stopEvent();
             return;
         }

         if ((message[0] == '/' || message[0] == '!') && message.length > 1) {
             if (parseInt(sys.time()) - lastMemUpdate > 500) {
                 lastMemUpdate = parseInt(sys.time());
             }
             print("Command -- " + sys.name(src) + ": " + message);

             sys.stopEvent();
             
             var command;
             var commandData;
             var pos = message.indexOf(' ');
	     var commandpart = new Array();

             if(chan==mafiachan){
                 try{
                     mafia.handleCommand(src,message.substr(1));
                     return; }
                 catch(err){
                     if(err!="no valid command"){
                         botEscapeAll("Error occurred: "+err,mafiachan);
                         mafia.endGame(0);
                         if(mafia.theme.name != "default") {
                             mafia.themeManager.disable(0,mafia.theme.name); }
                         return; } } } 
             if (pos != -1) {
                 command = message.substring(1, pos).toLowerCase();
                 commandData = message.substr(pos+1);
	         commandpart = commandData.split(':');
             } else {
                 command = message.substr(1).toLowerCase();
             }
             var tar = sys.id(commandData);

	     if (command == "commands" || command == "command"){
     	         sys.sendMessage(src, "+CommandBot: The commands are split into new commands - usercommands, modcommands, admincommands, and creatorcommands.");
		 return;
	     }
             if (command == "usercommands"){
                 sys.sendHtmlMessage(src, '');
                 sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** User Commands ***</font>');
                 sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** You can also use ! (exclamation mark) instead of / (front slash). Example: !rules ***</font>');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/rules</b></font> - to see the rules.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/me [action]</b></font> - to write an emote. Example: /me hides.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/emote [action]</b></font> - same as /me, except it does not use player color.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/die [message]</b></font> - disconnects you from the server with a message.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/players</b></font> - to get the number of players online.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/dice</b></font> - to roll two six-sided dice.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/claw</b></font> - to let you try your luck to get a prize out of a claw machine.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/coin</b></font><b> or </b><font color="green"><b>/flip</b></font> - to flip a coin to see if it is heads or tails.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/8ball [question]</b></font> - to receive an answer like an ordinary 8-Ball.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/ping [player]::[message]</b></font> - to get someone&#39;s attention with an optional message.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/disallowping [choice]</b></font> - to choose whether or not to disallow other players to ping you. Disabled by default. Recommended not to enable this!');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/faq</b></font> - to display frequently asked questions and answers to them.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/ranking</b></font> - to get your ranking in your tier.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/commands</b></font><b> or </b><font color="green"><b>/command</b></font> - to bring up the command categories.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/vote [choice]</b></font> - to vote in a poll.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/viewpoll</b></font> - to view the current poll, as well as the choices.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/join</b></font> - to join a tournament.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/leave</b></font><b> or </b><font color="green"><b>/leave</b></font> - to leave a tournament.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/updates [page]</b></font> - to view the updates of the server. They are separated into pages.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/viewround</b></font> - to view the pairings for the round.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/tourprize</b></font> - to see what the tournament prize is.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/spots</b></font> - to see the number of spots for the tournament remaining. Only works during signup phase.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/signbook [saying]</b></font> - to add a post to the guestbook with an optional saying.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/viewbook</b></font> - to view the guestbook so you can see the latest posts.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/viewmotd</b></font> - to view the current server notes that are displayed at login.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/viewtopic</b></font> - to view the channel topic.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/mutecheck</b></font> - to check to see if you are muted.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/massmutecheck</b></font> - to check to see if massmute is in effect.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/battlecheck</b></font> - to check to see if battles are enabled or not.');
                 sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/topic [text]</b></font> - to change the topic of a channel.');
	         return;
	     }
             if (command == "modcommands"){
                 if (sys.auth(src) < 1){
     	             sys.sendMessage(src, "+CommandBot: You are unable to view this list because you are not a moderator or above.");
                     return;
	         } else {
                     sys.sendHtmlMessage(src, '');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** Moderator Commands ***</font>');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** You can also use ! (exclamation mark) instead of / (front slash). Example: !rules ***</font>');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/newpoll Question:[choices]</b></font> - to start a poll. More choices are added with a frontslash.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/pollclose</b></font> - to close a poll.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/kick [player]:[reason]</b></font> to kick someone with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/silentkick [player]</b></font> - to kick someone without others knowing. Use at your own risk.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/mute [player]:[reason]</b></font> - to prevent someone from chatting with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/unmute [player]:[reason]</b></font> - to allow someone to chat with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/massmute</b></font> - to mute the chat, so no one can talk except for moderators and above. Type it again to unmute it.');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** Tournament Commands ***</font>');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/tour tier:number:prize</b></font> - to start a tier tournament consisting of number of players. The prize is optional.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/endtour</b></font> - to end the current tournament.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/dq [player]</b></font> - to disqualify someone in the tournament.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/changecount [entrants]</b></font> - to change the number of entrants during the signup phase.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/push [player]</b></font> - to add someone in the tournament.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/sub [player1]:[player2]</b></font> - to replace someone with someone else.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/cancelBattle [player1]</b></font> - to allow the player to forfeit their current battle so the player can battle their opponent again.');
	             return;
	         }
	     }
             if (command == "admincommands"){
                 if (sys.auth(src) < 2){
     	             sys.sendMessage(src, "+CommandBot: You are unable to view this list because you are not an administrator or above.");
                     return;
	         } else {
                     sys.sendHtmlMessage(src, '');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** Administrator Commands ***</font>');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** You can also use ! (exclamation mark) instead of / (front slash). Example: !rules ***</font>');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/ban [player]:[reason]</b></font> to ban someone with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/changeRating [player] -- [tier] -- [rating]</b></font> - to change the rating of someone.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/stopBattles</b></font> - to disable all new battles. When you want to close the server or do testing, use this command.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/reset</b></font> - to reset the server variables (useful when you add a new script).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/user [name]:[reason]</b></font> - to make someone into a user with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/mod [name]:[reason]</b></font> - to make someone into a moderator with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/admin [name]:[reason]</b></font> - to make someone into an administrator with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/clearbook</b></font> - to clear the guest book.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/showteam [someone]</b></font> - to help people who have problems with event moves or invalid teams.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/memorydump</b></font> - to see the state of the memory.');
	             return;
	         }
	     }
             if (command == "creatorcommands"){
                 if (sys.name(src) != "Roxas" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora"){
     	             sys.sendMessage(src, "+CommandBot: You are unable to view this list because the commands you are trying to view are only usable by the creator.");
		     return;
	         } else {
                     sys.sendHtmlMessage(src, '');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** Creator Commands ***</font>');
                     sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** You can also use ! (exclamation mark) instead of / (front slash). Example: !rules ***</font>');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/motd [message]</b></font> - to change the MOTD message.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/unban [player]</b></font> to unban someone.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/creatoruser [name]:[reason]</b></font> - to make someone into a user with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/creatormod [name]:[reason]</b></font> - to make someone into a moderator with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/creatoradmin [name]:[reason]</b></font> - to make someone into an administrator with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/creatorowner [name]:[reason]</b></font> - to make someone into a server owner with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/creatorbossify [name]:[reason]</b></font> - to make someone have auth rank 4 with a reason (optional).');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/serverpublic</b></font> - to make the server display on the registry.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/serverprivate</b></font> - to prevent the server from displaying on the registry.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/clearchat</b></font> - to clear the chat.');
                     sys.sendHtmlMessage(src, '<font color="green"><timestamp/> <b>/resetpass [name]</b></font> - to reset a player&#39;s password.');
                     return;
	         }
             }
             if (command == "me") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
                 if (message.length == 3)
                     return;
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
                     return;}
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         if (sys.getColor(src) == "#000000"){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> *** " + sys.name(src) + " " + commandData + " ***</font> ", channel);
	         } else {
	             sys.sendHtmlAll("<font color='" + sys.getColor(src) + "'><timestamp/> *** " + sys.name(src) + " " + commandData + " ***</font> ", channel);
                     return;
		 }
	     }
             if (command == "me's") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
                 if (message.length == 5)
                     return;
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
                     return;}
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         if (sys.getColor(src) == "#000000"){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> *** " + sys.name(src) + "&#39;s " + commandData + " ***</font> ", channel);
	         } else {
	             sys.sendHtmlAll("<font color='" + sys.getColor(src) + "'><timestamp/> *** " + sys.name(src) + "&#39;s " + commandData + " ***</font> ", channel);
                     return;
		 }
	     }
             if (command == "emote") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
                 if (message.length == 6)
                     return;
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
                     return;}
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         sys.sendAll("*** " + sys.name(src) + " " + commandData + " *** ", channel);
                 return;
	     }
             if (command == "emote's") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
                 if (message.length == 8)
                     return;
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
                     return;}
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         sys.sendAll("*** " + sys.name(src) + "'s " + commandData + " *** ", channel);
                 return;
	     }
             if (command == "updates") {
                 if (commandData == "1") {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "Updates, Page 1");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "(11/20/2010) Added /sendhtmlall command. It lets you send HTML code. Designed specifically for users and above.");
                     sys.sendMessage(src, "(11/9/2010) Renamed /silence to /massmute, and removed /silenceoff. It is a toggle command now. Again, still designed specifically for moderators and above.");
                     sys.sendMessage(src, "(11/9/2010) Merged /silencesec and /silencemin commands into /silence. Still designed specifically for moderators and above.");
                     sys.sendMessage(src, "(10/24/2010) Renamed /silence to /silencemin, and added /silencesec command. Designed specifically for moderators and above. It allows you to silence the chat for a number of seconds.");
                     sys.sendMessage(src, "(10/24/2010) Added /impcheck command. It allows you to check what or who you are impersonating without the need of saying something. Designed specifically for users and above.");
                     sys.sendMessage(src, "(10/23/2010) Split the Fifth_Gen tier into 2 tiers, Full_Dream_World and Dream_World.");
                     sys.sendMessage(src, "(10/15/2010) Added /mutecheck command. It allows you to check if you are muted. Designed specifically for users and above.");
                     sys.sendMessage(src, "(10/13/2010) It looks like Random Teams tier is not recommended for Generation 5, so it was removed.");
                     sys.sendMessage(src, "(10/9/2010) Added /silentkick command. It allows you to kick someone without others knowing. Designed specifically for moderators and above. Use at your own risk.");
                     sys.sendMessage(src, "(10/7/2010) Added /spots command. It allows you to see how many tournament spots are left.");
                     sys.sendMessage(src, "(10/7/2010) Replaced /changeauth with /user, /mod, /admin, and /owner commands, seeing /changeauth was a bit...problematic. Designed specifically for administrators and above.");
                     sys.sendMessage(src, "(10/4/2010) Removed /meon, /meoff, and all instances relating to those commands, seeing they were useless..");
                     sys.sendMessage(src, "(9/27/2010) Added Random Teams tier. You can use Shoddy Battle's 'Randomise Team' team builder feature to make a random team.");
                     sys.sendMessage(src, "(9/26/2010) Added /leave and /updates commands. Designed specifically for users and above.");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "END OF PAGE 1.");
                     sys.sendMessage(src, "");
                     return;
                 } else if (commandData == "2") {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "Updates, Page 2");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "(12/26/2010) Made /me use player color (Thanks, Kyu!). Also made /sendall, /sendhtmlall, /topic, and /me display in the channel it was posted in.");
                     sys.sendMessage(src, "(12/25/2010) Added /tourprize command. It lets you know what the prize of the tournament is. Designed for users and above.");
                     sys.sendMessage(src, "(12/25/2010) Added an optional prize for tournaments. They currently serve no use except for bragging rights.");
                     sys.sendMessage(src, "(12/17/2010) Added /dice command. It rolls 2 six-sided dice. Designed for users and above.");
                     sys.sendMessage(src, "(12/16/2010) Added message of the day, and /viewmotd and /motd commands. /viewmotd allows you to see the MOTD, and /motd changes it. /viewmotd is for users and above, and /motd is for administrators and above.");
                     sys.sendMessage(src, "(12/16/2010) Tiers were updated.");
                     sys.sendMessage(src, "(12/10/2010) Removed /imp, /impoff, and /impcheck commands as well as all traces of it, as people won't stop abusing them. Ever since the /sendhtmlall command, it's also been pretty outdated.");
                     sys.sendMessage(src, "(12/7/2010) Removed /eval command, as it was known to be dangerous..");
                     sys.sendMessage(src, "(12/7/2010) Removed original /kick, /mute, and /unmute commands, and renamed /kickreason, /mutereason, and /unmutereason commands to /kick, /mute, and /unmute, respectively. Those commands are still designed specifically for moderators and above.");
                     sys.sendMessage(src, "(12/7/2010) Updated /showteam command. Designed specifically for administrators and above.");
                     sys.sendMessage(src, "(12/7/2010) Added a tournament team check. It will have no effect on ChallengeCup tier, since it consists of random teams.");
                     sys.sendMessage(src, "(12/5/2010) Added /mutereason and /unmutereason commands. It's the same as mute and unmute commands, but allows for optional reason. Designed specifically for moderators and above.");
                     sys.sendMessage(src, "(12/5/2010) Updated the /command list to use HTML code.");
                     sys.sendMessage(src, "(12/5/2010) Added /kickreason command. It lets you kick someone with a reason (optional). Designed specifically for moderators and above.");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "END OF PAGE 2.");
                     sys.sendMessage(src, "");
	             return;
                 } else if (commandData == "3") {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "Updates, Page 3");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "(3/28/2011) Added /emote command. It is basically the same as /me, except it does not use player color; instead it uses the fuchsia color.");
                     sys.sendMessage(src, "(3/24/2011) Added /disallowping command. It lets you choose whether or not to disallow other players to ping you. It is disabled by default.");
                     sys.sendMessage(src, "(3/11/2011) Added /shoddyinfo command. It is separated into sections, read the sections so you know the stuff added to our Shoddy Battle server.");
                     sys.sendMessage(src, "(3/9/2011) Removed megauser because it was too faulty.");
                     sys.sendMessage(src, "(3/5/2011) Various updates and whatnot too numerous and difficult to mention.");
                     sys.sendMessage(src, "(3/4/2011) Added /unban command. It lets you unban a player with an optional reason. Designed for the creator of this server.");
                     sys.sendMessage(src, "(3/4/2011) Added /ban command. It lets you ban a player with an optional reason. Designed for administrators and above.");
                     sys.sendMessage(src, "(2/23/2011) Added /sendmsg and /sendhtmlmsg commands. Those commands are the same as /sendall and /sendhtmlall, but the message is sent to you instead of everyone else.");
                     sys.sendMessage(src, "(2/22/2011) Added /claw command. It lets you try to get a prize.");
                     sys.sendMessage(src, "(2/18/2011) Added /faq command. It displays the frequently asked questions and answers to them.");
                     sys.sendMessage(src, "(1/30/2011) Split /commands into new commands - /usercommands, /megacommands, /modcommands, /admincommands, and /creatorcommands.");
                     sys.sendMessage(src, "(1/30/2011) Separated /updates into pages.");
                     sys.sendMessage(src, "(1/30/2011) Added random greetings for players that enter the server.");
                     sys.sendMessage(src, "(1/17/2011) Added /signbook and /viewbook commands. They are commands that let you sign and view the guestbook respectively, so the MOTD makes more sense (Again, thanks, Kyu!). Designed for users and above.");
                     sys.sendMessage(src, "(1/16/2011) Added /viewtopic command. It lets you view the channel topic. Designed for users and above.");
                     sys.sendMessage(src, "(1/11/2011) Added /8ball command. It is yet another fun command that acts like an ordinary 8-Ball. Designed for users and above.");
                     sys.sendMessage(src, "(1/10/2011) Added reasons for /megauser and /megauseroff commands. Still designed for administrators and above.");
                     sys.sendMessage(src, "(1/8/2011) Added reasons for /user, /mod, /admin, and /owner commands. Still designed for administrators and above.");
                     sys.sendMessage(src, "(1/6/2011) Added /ping command. It just seems like another way to get someone's attention. Designed for users and above.");
                     sys.sendMessage(src, "(1/4/2011) Added an alternate way of /coin command - /flip.");
                     sys.sendMessage(src, "(1/2/2011) Added /coin command. It flips a coin that checks if it's heads or tails. Can be used for a game of chance or just plain fun. Designed for users and above.");
                     sys.sendMessage(src, "(1/1/2011) Made /motd use only for the creator of this server. Also changed /motd command section from admin to name-specific, which was also added.");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "END OF PAGE 3.");
                     sys.sendMessage(src, "");
                     return;
                 } else if (commandData == "4") {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "Updates, Page 4");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "(12/10/2011) Removed /shoddyinfo and all traces of it because of failure to recover the Shoddy Battle server.");
                     sys.sendHtmlMessage(src, '<timestamp/> (11/7/2011) Added full HTML support for /ping. Also functions as /ping [player]::[message] to prevent conflicts with messages with the : symbol.');
                     sys.sendMessage(src, "(9/9/2011) Updated /showteam command. Still designed for administrator and above.");
                     sys.sendMessage(src, "(9/1/2011) Added /die command. This one is basically what you expect. Worked on by vappy.");
                     sys.sendMessage(src, "(8/4/2011) Added another way to leave the tournament - /leave.");
                     sys.sendMessage(src, "(6/6/2011) Added /serverprivate and /serverpublic commands. /serverprivate prevents the server from displaying on the registry while /serverpublic allows the server to display on the registry. Designed for the server creators.");
                     sys.sendMessage(src, "(4/29/2011) Made /ping command support messages.");
                     sys.sendMessage(src, "(4/27/2011) Added /massmutecheck command. It lets you display the current status of massmute.");
                     sys.sendMessage(src, "(4/9/2011) Added a massmute reminder that reminds players that join in about the current status of massmute.");
                     sys.sendMessage(src, "(4/7/2011) Added /viewpoll command. It lets you view the currently running poll, as well as the choices.");
                     sys.sendMessage(src, "(4/3/2011) Added authority level changing commands for the creator of this server. Also changed the admin commands so it does not change the authority level of the creator of this server.");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "END OF PAGE 4.");
                     sys.sendMessage(src, "");
                     return;
                 } else if (commandData == "5") {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "Updates, Page 5");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "(1/9/2012) Removed /setpa. (What purpose did that have, anyway?)");
                     sys.sendMessage(src, "(1/29/2013) Added /battlecheck command. It allows you to check if battles are enabled or disabled.");
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "END OF PAGE 5.");
                     sys.sendMessage(src, "");
                     return;
                 } else {
	             sys.sendMessage(src, "Please select a page. There are currently 5 pages. Example - '/updates 1' displays page 1.");
	             return;
	         }
	     }
             if (command == "viewmotd") {
	         sys.sendHtmlMessage(src, "<timestamp/> <b>Message of the Day:</b> " + MOTD);
                 return;
             }
             if (command == "viewtopic") {
                 if (channel == 0) {
	             sys.sendMessage(src, "+TopicBot: There is no topic in the main channel.", channel);
	             return;
	         } else if (channelTopics[chan] == undefined) {
	             sys.sendHtmlMessage(src, "<timestamp/> <b>Channel topic:</b> No topic is set.", channel);
                     return;
                 } else {
	             sys.sendHtmlMessage(src, "<timestamp/> <b>Channel topic:</b> " + channelTopics[chan], channel);
                     return;
                 }
             }
	     if (command == "viewpoll") { // Astruvis' Command
		 if (!server.pollmode) {
		     sys.sendMessage(src,'+Bot: Error, no poll open.');
		     return; }
		 sys.sendMessage(src, '+Bot: Poll; ' + pollstuff);
		 sys.sendMessage(src, '+Bot: Please vote! Use /vote option');
		 for ( var z in server.polloptions ) {
		     sys.sendMessage(src, '+Bot: Option #'+(z*1+1)+': '+ server.polloptions[z]); }
		 return; }
             if (command == "spots") {
                 if (tourmode != 1){
                     sys.sendMessage(src, "That command only works during signup phase.");
                     return;
                 }
                 sys.sendHtmlMessage(src, "<timestamp/> A tournament (" + tourtier + ") was started with " + tournumber +" players. Spots left: " +this.tourSpots() + ".");
                 return;
             }
             if (command == "massmutecheck") {
                 if (muteall) {
                     sys.sendMessage(src, "+MuteBot: Massmute is currently in effect.");
		     return;
                 } else {
                     sys.sendMessage(src, "+MuteBot: Massmute is currently not in effect.");
		     return;
                 }
             }
             if (command == "battlecheck") {
                 if (battlesStopped) {
                     sys.sendMessage(src, "+BattleBot: Battles are currently disabled.");
		     return;
                 } else {
                     sys.sendMessage(src, "+BattleBot: Battles are currently enabled.");
		     return;
                 }
             }
	     if (command == "signbook"){
                 if (commandData == undefined) {
		     var get = sys.getFileContent("gb");
		     sys.writeToFile("gb",get + sys.name(src) + "::");
		     sys.sendAll(sys.name(src) + " " + "signed the guest book!");
                     return;
                 }
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
		 var get = sys.getFileContent("gb");
		 sys.writeToFile("gb",get + sys.name(src) + " - " + commandData + "::");
		 sys.sendAll(sys.name(src) + " " + "signed the guest book!");
		 return;
	     }
	     if (command == "viewbook"){
                 sys.sendHtmlMessage(src, "");
                 sys.sendHtmlMessage(src, '<font color="blue"><timestamp/> *** The guest book ***</font>');
                 sys.sendHtmlMessage(src, "");
	         var get = sys.getFileContent("gb").split("::");
		 for(x in get){
		     sys.sendHtmlMessage(src, get[x]);
		 }
		 return;
	     }
             if (command == "tourprize") {
                 if (tourmode == 1 && prize == undefined){
                     sys.sendMessage(src, "The prize of the tournament is nothing.");
                     return;
                 } else if (tourmode == 1){
	             sys.sendMessage(src, "The prize of the tournament is " + prize);
		     return;
                 } else if (tourmode == 2 && prize == undefined){
                     sys.sendMessage(src, "The prize of the tournament is nothing.");
                     return;
                 } else if (tourmode == 2){
	             sys.sendMessage(src, "The prize of the tournament is " + prize);
		     return;
	         } else {
	             sys.sendMessage(src, "There is no tournament as of now.");
                     return;
	         }
             }
             if (command == "mutecheck") {
                 if (channel != mafiachan && muted[src] === true){
                     sys.sendMessage(src, "You are muted.");
                     return;
                 }
                 sys.sendMessage(src, "You are not muted.");
                 return;
             }
	     if (command == "ping") {
                 commandpart = commandData.split('::');
		 var tar = sys.id(commandpart[0])
		 var nametar = sys.name(tar);
		 var namesrc = sys.name(src);
		 pingmessage = commandpart[1]
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         pingmessage+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         if (pingstatus[tar] == true) {
	             sys.sendMessage(src, "+Bot: That player does not allow to be pinged.");
	             sys.stopEvent();
	             return;
	         }
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (pingmessage == undefined) {
                     sys.sendHtmlMessage(src, "<font color='#3DAA68'><timestamp/> <b>+Bot:</b></font> You have pinged " + nametar + ".");
	             sys.sendHtmlMessage(tar, "<font color='#3DAA68'><ping/><timestamp/> <b>+Bot:</b></font> " + namesrc + " is pinging you.");
                     return;
	         }
                 sys.sendHtmlMessage(src, "<font color='#3DAA68'><timestamp/> <b>+Bot:</b></font> You have pinged " + nametar + ". Message: " + pingmessage);
	         sys.sendHtmlMessage(tar, "<font color='#3DAA68'><ping/><timestamp/> <b>+Bot:</b></font> " + namesrc + " is pinging you. Message: " + pingmessage);
                 return;
             }
             if (command == "rules") {
                 for (rule in rules) {
                     sys.sendMessage(src, rules[rule]);
                 }
                 return;
             }
             if (command == "faq") {
	         sys.sendMessage(src, "Frequently Asked Questions");
	         sys.sendMessage(src, "");
	         sys.sendMessage(src, "Question 1: How come I cannot battle?");
	         sys.sendMessage(src, "Answer 1: It is most likely because you do not have a team. Just make a team with any Pokemon of your choosing and you should be all set. Do not forget to add moves to your Pokemon, or they will be unusable.");
	         sys.sendMessage(src, "");
	         sys.sendMessage(src, "Question 2: How do I join a tournament?");
	         sys.sendMessage(src, "Answer 2: Make sure your tier is the same as the tournament's, then type /join. To leave the tournament, you use /leave.");
	         sys.sendMessage(src, "");
	         sys.sendMessage(src, "Question 3: Why does the server crash every now and then?");
	         sys.sendMessage(src, "Answer 3: The answer is currently unknown.");
	         sys.sendMessage(src, "");
	         sys.sendMessage(src, "Question 4: How come I cannot use certain commands?");
	         sys.sendMessage(src, "Answer 4: One of the reasons being you have an insufficient authority level. /creatorcommands and the commands that fall under that category were designed for the creators of the server, not server owners.");
	         sys.sendMessage(src, "");
	         sys.sendMessage(src, "END OF FAQ");
                 return;
             }
             if (command == "players") {
                 sys.sendMessage(src, "+CountBot: Number of players online is " + sys.numPlayers() + ". High score " + sys.getVal("MaxPlayersOnline") + ".");
                 return;
             }
             if (command == "dice") {
	         diceresult = sys.rand(1, 7)
	         diceresulta = sys.rand(1, 7)
                 sys.sendMessage(src, "+DiceBot: You rolled a " + diceresult + " and a " + diceresulta + ".");
                 return;
             }
             if (command == "coin" || command == "flip") {
	         coinresult = sys.rand(1, 3)
	         if (coinresult == 1){
                     sys.sendMessage(src, "+FlipBot: You flipped a coin and got heads.");
	         } else {
                     sys.sendMessage(src, "+FlipBot: You flipped a coin and got tails.");
                     return;
                 }
             }
	     if (command == "die") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
	         diemsg = commandData
	         deathresult = sys.rand(1, 12)
	         if (diemsg == undefined && deathresult == 1){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " died. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 2){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " should have read the warning label. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 3){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " messed with mastur ch33f and got shot with a shotgun. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 4){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " boldly went where no player has gone before. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 5){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " was eaten by the exit. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 6){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " learned why you don't shoot down a Cinccino's (spaceship) ride. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 7){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " tried to jump the flagpole (like Mario) but failed. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 8){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " lost a game of dodge-tail-slap with a Minccino. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 9){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " learned that spilling hot coffee on a Minccino is bad. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " was teleported out. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         }
	         sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " " + diemsg + " ===</font> ", channel);
	         sys.setTimer("sys.kick(" + src + ")", 1, 0);
                 return;
             }
	     if (command == "die's") {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
	         diemsg = commandData
	         deathresult = sys.rand(1, 12)
	         if (diemsg == undefined && deathresult == 1){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " died. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 2){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " should have read the warning label. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 3){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " messed with mastur ch33f and got shot with a shotgun. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 4){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " boldly went where no player has gone before. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 5){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " was eaten by the exit. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 6){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " learned why you don't shoot down a Cinccino's (spaceship) ride. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 7){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " tried to jump the flagpole (like Mario) but failed. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 8){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " lost a game of dodge-tail-slap with a Minccino. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined && deathresult == 9){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " learned that spilling hot coffee on a Minccino is bad. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         } else if (diemsg == undefined){
	             sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + " was teleported out. ===</font> ", channel);
	             sys.setTimer("sys.kick(" + src + ")", 1, 0);
                     return;
	         }
	         sys.sendHtmlAll("<font color='fuchsia'><timestamp/> === " + sys.name(src) + "'s " + diemsg + " ===</font> ", channel);
	         sys.setTimer("sys.kick(" + src + ")", 1, 0);
                 return;
             }
             if (command == "claw") {
	         clawresult = sys.rand(1, 106)
	         if (clawresult == 1){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 2){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Shinku doll! Ow! She kicked you in the face!");
	         } else if (clawresult == 3){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Shinku doll! Ow! She kicked you in the face!");
	         } else if (clawresult == 4){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Shinku doll! Ow! She kicked you in the face!");
	         } else if (clawresult == 5){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Smash Ball. Use it quickly!");
	         } else if (clawresult == 6){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Smash Ball. Use it quickly!");
	         } else if (clawresult == 7){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Smash Ball. Use it quickly!");
	         } else if (clawresult == 8){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Clefairy. It runs back into the claw machine.");
	         } else if (clawresult == 9){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Clefairy. It runs back into the claw machine.");
	         } else if (clawresult == 10){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a shiny Clefairy! Sadly, it runs back into the claw machine.");
	         } else if (clawresult == 11){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush shiny Clefairy!");
	         } else if (clawresult == 12){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Smeargle figure.");
	         } else if (clawresult == 13){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Smeargle figure.");
	         } else if (clawresult == 14){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Cleffa.");
	         } else if (clawresult == 15){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Cleffa.");
	         } else if (clawresult == 16){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a brand new bag of popcorn. Wait, that's not a snack machine!");
	         } else if (clawresult == 17){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a burnt out lightbulb. What use is this for?");
	         } else if (clawresult == 18){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a can of mushrooms. You are probably wondering who puts food in the claw machine.");
	         } else if (clawresult == 19){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Mew. You must be pretty happy.");
	         } else if (clawresult == 20){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a water bottle. Great, a claw machine that gives out refreshments.");
	         } else if (clawresult == 21){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a bucket full of colored Yoshi figures.");
	         } else if (clawresult == 22){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a live trout that flops away into a nearby lake.");
	         } else if (clawresult == 23){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Jigglypuff.");
	         } else if (clawresult == 24){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Jigglypuff.");
	         } else if (clawresult == 25){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Pikachu. For some reason, despite being really big, you grab it with ease.");
	         } else if (clawresult == 26){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Pikachu. For some reason, despite being really big, you grab it with ease.");
	         } else if (clawresult == 27){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Reuniclus.");
	         } else if (clawresult == 28){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Reuniclus.");
	         } else if (clawresult == 29){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Minccino.");
	         } else if (clawresult == 30){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Minccino.");
	         } else if (clawresult == 31){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Cinccino.");
	         } else if (clawresult == 32){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Cinccino.");
	         } else if (clawresult == 33){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a small claw machine. It looks exactly the same as the one you played.");
	         } else if (clawresult == 34){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Wailord. You are able to get it out of the claw machine with ease.");
	         } else if (clawresult == 35){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Azurill.");
	         } else if (clawresult == 36){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Azurill.");
	         } else if (clawresult == 37){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a pizza. Delicious, yet strange.");
	         } else if (clawresult == 38){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a pizza. Delicious, yet strange.");
	         } else if (clawresult == 39){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Bulbasaur.");
	         } else if (clawresult == 40){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Charmander.");
	         } else if (clawresult == 41){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Squirtle.");
	         } else if (clawresult == 42){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Pikachu.");
	         } else if (clawresult == 43){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Chikorita.");
	         } else if (clawresult == 44){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Cyndaquil.");
	         } else if (clawresult == 45){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Totodile.");
	         } else if (clawresult == 46){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Treecko.");
	         } else if (clawresult == 47){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Torchic.");
	         } else if (clawresult == 48){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Mudkip.");
	         } else if (clawresult == 49){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Turtwig.");
	         } else if (clawresult == 50){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Chimchar.");
	         } else if (clawresult == 51){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Piplup.");
	         } else if (clawresult == 52){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Snivy.");
	         } else if (clawresult == 53){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Tepig.");
	         } else if (clawresult == 54){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Oshawott.");
	         } else if (clawresult == 55){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Eevee.");
	         } else if (clawresult == 56){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Gothitelle.");
	         } else if (clawresult == 57){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 58){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 59){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 60){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 61){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 62){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 63){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
	         } else if (clawresult == 64){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a Volcarona. It flies away into the sun.");
	         } else if (clawresult == 65){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a shiny Volcarona! Sadly, it flies away into the sun.");
	         } else if (clawresult == 66){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Ivysaur.");
	         } else if (clawresult == 67){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Charmeleon.");
	         } else if (clawresult == 68){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Wartortle.");
	         } else if (clawresult == 69){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Raichu.");
	         } else if (clawresult == 70){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Bayleef.");
	         } else if (clawresult == 71){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Quilava.");
	         } else if (clawresult == 72){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Croconaw.");
	         } else if (clawresult == 73){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Grovyle.");
	         } else if (clawresult == 74){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Combusken.");
	         } else if (clawresult == 75){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Marshtomp.");
	         } else if (clawresult == 76){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Grotle.");
	         } else if (clawresult == 77){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Monferno.");
	         } else if (clawresult == 78){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Prinplup.");
	         } else if (clawresult == 79){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Servine.");
	         } else if (clawresult == 80){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Pignite.");
	         } else if (clawresult == 81){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Dewott.");
	         } else if (clawresult == 82){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Venusaur.");
	         } else if (clawresult == 83){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Charizard.");
	         } else if (clawresult == 84){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Blastoise.");
	         } else if (clawresult == 85){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Pichu.");
	         } else if (clawresult == 86){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Meganium.");
	         } else if (clawresult == 87){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Typhlosion.");
	         } else if (clawresult == 88){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Feraligatr.");
	         } else if (clawresult == 89){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Sceptile.");
	         } else if (clawresult == 90){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Blaziken.");
	         } else if (clawresult == 91){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Swampert.");
	         } else if (clawresult == 92){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Torterra.");
	         } else if (clawresult == 93){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Infernape.");
	         } else if (clawresult == 94){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Empoleon.");
	         } else if (clawresult == 95){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Serperior.");
	         } else if (clawresult == 96){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Emboar.");
	         } else if (clawresult == 97){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Samurott.");
	         } else if (clawresult == 98){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Vaporeon.");
	         } else if (clawresult == 99){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Flareon.");
	         } else if (clawresult == 100){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Jolteon.");
	         } else if (clawresult == 101){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Espeon.");
	         } else if (clawresult == 102){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Umbreon.");
	         } else if (clawresult == 103){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Leafeon.");
	         } else if (clawresult == 104){
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get a plush Glaceon.");
	         } else {
                     sys.sendMessage(src, "+ClawBot: You play the claw machine and get nothing.");
                     return;
                 }
             }
             if (command == "8ball") {
	         ballanswer = sys.rand(1, 18)
	         if (ballanswer == 1){
                     sys.sendMessage(src, "+8Ball: It seems likely that will happen.");
	         } else if (ballanswer == 2){
                     sys.sendMessage(src, "+8Ball: It seems unlikely that will happen.");
	         } else if (ballanswer == 3){
                     sys.sendMessage(src, "+8Ball: Statistics indicate yes.");
	         } else if (ballanswer == 4){
                     sys.sendMessage(src, "+8Ball: Statistics indicate no.");
	         } else if (ballanswer == 5){
                     sys.sendMessage(src, "+8Ball: Outlook good.");
	         } else if (ballanswer == 6){
                     sys.sendMessage(src, "+8Ball: Outlook not so good.");
	         } else if (ballanswer == 7){
                     sys.sendMessage(src, "+8Ball: Without a doubt.");
	         } else if (ballanswer == 8){
                     sys.sendMessage(src, "+8Ball: Ask again later.");
	         } else if (ballanswer == 9){
                     sys.sendMessage(src, "+8Ball: Ask again in the spring.");
	         } else if (ballanswer == 10){
                     sys.sendMessage(src, "+8Ball: Ask again in the summer.");
	         } else if (ballanswer == 11){
                     sys.sendMessage(src, "+8Ball: Ask again in the fall.");
	         } else if (ballanswer == 12){
                     sys.sendMessage(src, "+8Ball: Ask again in the winter.");
	         } else if (ballanswer == 13){
                     sys.sendMessage(src, "+8Ball: Go for it!");
	         } else if (ballanswer == 14){
                     sys.sendMessage(src, "+8Ball: Definitely.");
	         } else if (ballanswer == 15){
                     sys.sendMessage(src, "+8Ball: It is certain.");
	         } else if (ballanswer == 16){
                     sys.sendMessage(src, "+8Ball: There is a high chance of that.");
	         } else {
                     sys.sendMessage(src, "+8Ball: Outlook vague; try again.");
                     return;
                 }
             }
                          if (command == "sendmsg") {
                 sys.sendMessage(src, commandData, channel);
                 return;
             }
	     if (command == 'sendhtmlall') {
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
	             return;
	         } else {
		     sys.sendHtmlAll('<timestamp/>' +commandData, channel); 
		     return; 
	         }
             }
	     if (command == 'sendhtmlmsg') {
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
		 sys.sendHtmlMessage(src, '<timestamp/>' +commandData, channel); }
             if (command == "ranking") {
                 for (var team = 0; team < sys.teamCount(src); team++) {
                     var rank = sys.ranking(src, team);
                     if (rank == undefined) {
                         sys.sendMessage(src, "+RankingBot: You are not ranked in " + sys.tier(src, team) + " yet!");
                     } else {
                         sys.sendMessage(src, "+RankingBot: Your rank in " + sys.tier(src, team) + " is " + rank + "/" + sys.totalPlayersByTier(sys.tier(src, team)) + "!");
                     }
                     return;
                 }
             }
             if (command == "topic") {
                 if (commandData == undefined) {
                     sys.sendMessage(src, "+Bot: Specify a topic!");
                     return;
                 }
                 if (channel == 0) {
                     sys.sendMessage(src, "+Bot: You can't do that in main channel.");
                     return;
                 }
                 if (sys.auth(src) == 0 && (typeof(channelUsers[chan]) == 'undefined' || channelUsers[chan] != src)) {
                     sys.sendMessage(src, "+Bot: You don't have the rights.");
                     return;
                 }
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
	         if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
                 channelTopics[chan] = commandData;
                 sys.sendHtmlAll('<font color="#3DAA68"><timestamp/> <b>+ChannelBot:</b></font> ' + sys.name(src) + ' changed the topic to: ' + commandData, channel);
                 return;
             }
             if (command == "disallowping") {
                 if (commandData == "yes"){
                     sys.sendMessage(src, "+Bot: You do not allow other players to ping you.", chan);
                 } else if (commandData == "no") {
                     sys.sendMessage(src, "+Bot: You allow other players to ping you.", chan);
	         } else {
		     sys.sendMessage(src, "+Bot: Please choose whether or not to disallow ping with yes or no.", chan);
		     return;
	         }
                 pingstatus[src] = commandData == "yes";
                 saveKey("pingstatus", src, pingstatus[src] * 1);
                 return;
             }
             if (command == "join"){
                 for (var team = 0; team < sys.teamCount(src); team++) {
                     if (channel != 0) {
                         sys.sendMessage(src, "+TourBot: You must be in the main channel to join a tournament!");
                         return;
                     }
                     if (tourmode != 1){
                         sys.sendMessage(src, "Sorry, you are unable to join because a tournament is not currently running or has passed the signups phase.");
                         return;
                     }
                     var name = sys.name(src).toLowerCase();
                     if (tourmembers.indexOf(name.toLowerCase()) != -1){
                         sys.sendMessage(src, "Sorry, you are already in the tournament. You are not able to join more than once.");
                         return;
                     }
                     var srctier = sys.tier(src, team);
                     if (!cmp(srctier, tourtier)){
                         sys.sendMessage(src, "You are currently not battling in the " + tourtier + " tier. Change your tier to " + tourtier + " to be able to join.");
                         return;
                     }
                     if (this.tourSpots() > 0){
                         tourmembers.push(name);
                         tourplayers[name] = sys.name(src);
                         sys.sendAll("+Bot: " + sys.name(src) + " joined the tournament!", 0);
                         sys.sendAll("+Bot: Spots left: " + this.tourSpots() + ".", 0);
                         if (this.tourSpots() == 0){
                             tourmode = 2;
                             roundnumber = 0;
                             this.roundPairing();
                         }
                     }    
                     return;
                 }
             }
             if (command == "leave" || command == "leave"){
                 if (channel != 0) {
                     sys.sendMessage(src, "+TourBot: You must be in the main channel to leave a tournament!");
                     return;
                 }
                 var name = sys.name(src).toLowerCase();
                 if (tourmembers.indexOf(name.toLowerCase()) == -2){
                     sys.sendMessage(src, "It seems you already left or you did not join at all. There may not even be a tournament going on.");
                     return;
                 }
                 if (tourmode == 0) {
                     sys.sendMessage(src, "Sorry, you are unable to leave because a tournament is not currently running.");
                     return;
                 }
                 if (tourmembers.indexOf(name) != -1) {
                     tourmembers.splice(tourmembers.indexOf(name),1);
                     tourplayers[name] = sys.name(src);
                     delete tourplayers[name];
                     sys.sendAll("+Bot: " + sys.name(src) + " left the tournament!", 0);
                     return;
                 }
                 if (tourbattlers.indexOf(name) != -1) {
                     battlesStarted[Math.floor(tourbattlers.indexOf(name)/2)] = true;
                     sys.sendAll("+Bot: " + sys.name(src) + " left the tournament!", 0);
                     this.tourBattleEnd(this.tourOpponent(name), name);
                 }
                 return;
             }
	     
	     if (command == "vote") { // Astruvis' Command
		 if (!server.pollmode) {
		     sys.sendMessage(src,'+Bot: Error, no poll open.');
		     return; }
		 if (isNaN(commandpart[0]*1)) {
		     sys.sendMessage(src,'+Bot: Error, please use /vote option (where option is a number.)');
		     return; }
		 server.pollvotes[sys.ip(src)] = commandpart[0]*1;
		 sys.sendMessage(src,'+Bot: Your vote was added!'); 
		 return; }
	     
             if (command == "viewround"){
                 if (tourmode != 2){
                     sys.sendMessage(src, "Sorry, you are unable to view the round because a tournament is not currently running or is in signing up phase.");
                     return;
                 }
                 
                 var finales = tourmembers.length == 2;

                 if (finales) {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, border);
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "*** Viewing status of current round of " + tourtier + " tournament ***", 0);
                 }
                 else {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, border);
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "*** Viewing status of " + tourtier + " tournament (Round " + roundnumber + ") ***", 0);
                 }
                 
                 if (battlesLost.length > 0) {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "*** Battles finished ***");
                     sys.sendMessage(src, "");
                     for (var i = 0; i < battlesLost.length; i+=2) {
                         sys.sendMessage(src, battlesLost[i] + " won against " + battlesLost[i+1]);
                     }
                     sys.sendMessage(src, "");
                 }
                 
                 if (tourbattlers.length > 0) {
                     if (battlesStarted.indexOf(true) != -1) {
                         sys.sendMessage(src, "");
                         sys.sendMessage(src, "*** Ongoing battles ***");
                         sys.sendMessage(src, "");
                         for (var i = 0; i < tourbattlers.length; i+=2) {
                             if (battlesStarted [i/2] == true)
                                 sys.sendMessage(src, this.padd(tourplayers[tourbattlers[i]]) + " VS " + tourplayers[tourbattlers[i+1]]);
                         }
                         sys.sendMessage(src, "");
                     }
                     if (battlesStarted.indexOf(false) != -1) {
                         sys.sendMessage(src, "");
                         sys.sendMessage(src, "*** Yet to start battles ***");
                         sys.sendMessage(src, "");
                         for (var i = 0; i < tourbattlers.length; i+=2) {
                             if (battlesStarted [i/2] == false)
                                 sys.sendMessage(src, tourplayers[tourbattlers[i]] + " VS " + tourplayers[tourbattlers[i+1]]);
                         }
                         sys.sendMessage(src, "");
                     }
                 }
                 
                 if (tourmembers.length > 0) {
                     sys.sendMessage(src, "");
                     sys.sendMessage(src, "*** Members to the next round ***");
                     sys.sendMessage(src, "");
                     var str = "";
                     
                     for (x in tourmembers) {
                         str += (str.length == 0 ? "" : ", ") + tourplayers[tourmembers[x]];
                     }
                     sys.sendMessage(src, str);
                     sys.sendMessage(src, "");
                 }
                 
                 sys.sendMessage(src, border);
                 sys.sendMessage(src, "");
                 
                 return;
             }

	     /** Nowhere else to put this.. **/
             if (command == "motd") {
                 if (sys.name(src) != "Roxas" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to change the MOTD.");
                     return;
                 }
                 if (commandData == undefined) {
                     sys.sendMessage(src, "+Bot: Specify a message!");
                     return;
                 }
		 var qu = commandData.match(/<(\w+)[^>]*>/g);
		 if (qu) {
		     for (var x in qu) {
		         commandData+= qu[x].replace(/<(\w+)[^>]*>/g,'</$1>'); }
		 }
                 var m = commandData.toLowerCase();
                 if (m.indexOf("&#8238;") != -1 || m.indexOf("&#x202e;") != -1 || m.indexOf("&#x0E49;") != -1 || m.indexOf("&#3657;") != -1) {
                     sys.stopEvent();
                     return;
                 }
	         if (message.search(/[\u202E\u202D]/) != -1) {
		     return; }
	         sys.saveVal("MOTD", commandData)
                 MOTD = commandData;
                 sys.sendHtmlAll('<font color="#3DAA68"><timestamp/> <b>+MessageBot:</b></font> ' + sys.name(src) + ' changed the MOTD to: ' + commandData);
                 return;
             }
	     if (command == "creatoruser") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to use this command, use /user instead.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a user by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 0);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a user by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 0);
                 return;
             }
	     if (command == "creatormod") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to use this command, use /mod instead.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a moderator by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 1);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a moderator by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 1);
                 return;
             }
	     if (command == "creatoradmin") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to use this command, use /admin instead.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became an administrator by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 2);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became an administrator by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 2);
                 return;
             }
	     if (command == "creatorowner") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to use this command, use /owner instead.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a server owner by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 3);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a server owner by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 3);
                 return;
             }
	     if (command == "creatorbossify") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to use this command.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
                     sys.changeAuth(tar, 4);
                     return;
	         }
                 sys.changeAuth(tar, 4);
                 return;
             }

	     /** Now for the moderator commands because megauser is faulty.. **/
             if (sys.auth(src) < 1) {
                 return;
             }
             if (command == "dq") {
                 if (tourmode == 0) {
                     sys.sendMessage(src, "+TourneyBot: Wait until the tournament has started.");
                     return;
                 }
                 var name2 = commandData.toLowerCase();
                 
                 if (tourmembers.indexOf(name2) != -1) {
                     tourmembers.splice(tourmembers.indexOf(name2),1);
                     delete tourplayers[name2];
                     sys.sendAll("+TourneyBot: " + commandData + " was removed from the tournament by " + sys.name(src) + "!", 0);
                     return;
                 }
                 if (tourbattlers.indexOf(name2) != -1) {
                     battlesStarted[Math.floor(tourbattlers.indexOf(name2)/2)] = true;
                     sys.sendAll("+TourneyBot: " + commandData + " was removed from the tournament by " + sys.name(src) + "!", 0);
                     this.tourBattleEnd(this.tourOpponent(name2), name2);
                 }
                 return;
             }
             if (command == "push") {
                 if (tourmode == 0) {
                     sys.sendMessage(src, "+TourneyBot: Wait until the tournament has started.");
                     return;
                 }
                 if (this.isInTourney(commandData.toLowerCase())) {
                     sys.sendMessage(src, "+TourneyBot: " +commandData + " is already in the tournament.");
                     return;
                 }
                 sys.sendAll("+TourneyBot: " +commandData + " was added to the tournament by " + sys.name(src) + ".", 0);
                 tourmembers.push(commandData.toLowerCase()); 
                 tourplayers[commandData.toLowerCase()] = commandData;
                 
                 if (tourmode == 1 && this.tourSpots() == 0) {
                     tourmode = 2;
                     roundnumber = 0;
                     this.roundPairing();
                 }
                 return;
             }
             if (command == "cancelbattle") {
                 if (tourmode != 2) {
                     sys.sendMessage(src, "Wait until a tournament starts.");
                     return;
                 }
                 var name = commandData.toLowerCase();
                 
                 if (tourbattlers.indexOf(name) != -1) {
                     battlesStarted[Math.floor(tourbattlers.indexOf(name)/2)] = false;
                     sys.sendAll("+TourBot: " + commandData + " can forfeit their battle and rematch now by " + sys.name(src) + ".", 0);
                 }
                 
                 return;
             }
             if (command == "sub") {
                 if (tourmode != 2) {
                     sys.sendMessage(src, "Wait until a tournament starts.");
                     return;
                 }
                 var players = commandData.split(':');
                 
                 if (!this.isInTourney(players[0]) && !this.isInTourney(players[1])) {
                     sys.sendMessage(src, "+TourBot: Neither are in the tourney.");
                     return;
                 }
                 sys.sendAll("+TourBot: " + players[0] + " and " + players[1] + " were exchanged places in the ongoing tournament by " + sys.name(src) + ".", 0);
                 
                 var p1 = players[0].toLowerCase();
                 var p2 = players[1].toLowerCase();
                 
                 for (x in tourmembers) {
                     if (tourmembers[x] == p1) {
                         tourmembers[x] = p2;
                     } else if (tourmembers[x] == p2) {
                         tourmembers[x] = p1;
                     }
                 }
                 for (x in tourbattlers) {
                     if (tourbattlers[x] == p1) {
                         tourbattlers[x] = p2;
                         battlesStarted[Math.floor(x/2)] = false;
                     } else if (tourbattlers[x] == p2) {
                         tourbattlers[x] = p1;
                         battlesStarted[Math.floor(x/2)] = false;
                     }
                 }
                 
                 if (!this.isInTourney(p1)) {
                     tourplayers[p1] = players[0];
                     delete tourplayers[p2];
                 } else if (!this.isInTourney(p2)) {
                     tourplayers[p2] = players[1];
                     delete tourplayers[p1];
                 }
                 
                 return;
             }
             if (command == "tour"){
                 if (typeof(tourmode) != "undefined" && tourmode > 0){
                     sys.sendMessage(src, "Sorry, you are unable to start a tournament because one is still currently running.");
                     return;
                 }
                 
                 if (commandData.indexOf(':') == -1)
                     commandpart = commandData.split(' ');
                 else
                     commandpart = commandData.split(':');
                 
                 tournumber = parseInt(commandpart[1]);
	         prize = commandpart[2]
                 
                 if (isNaN(tournumber) || tournumber <= 2){                        
                     sys.sendMessage(src, "You must specify a tournament size of 3 or more.");
                     return;
                 }
                 
                 var tier = sys.getTierList();
                 var found = false;
                 for (var x in tier) {
                     if (cmp(tier[x], commandpart[0])) {
                         tourtier = tier[x];
                         found = true;
                         break;
                     }
                 }
                 if (!found) {
                     sys.sendMessage(src, "Sorry, the server does not recognise the " + commandpart[0] + " tier.");
                     return;
                 }

                 tourmode = 1;
                 tourmembers = [];
                 tourbattlers = [];
                 tourplayers = [];
                 battlesStarted = [];
                 battlesLost = [];

	         if (prize == undefined) {
                     sys.sendAll("", 0);
                     sys.sendAll(border, 0);
                     sys.sendAll("*** A tournament was started by " + sys.name(src) + "! ***", 0);
                     sys.sendAll("PLAYERS: " + tournumber, 0);
                     sys.sendAll("TIER: " + tourtier, 0);
                     sys.sendAll("PRIZE: None", 0);
                     sys.sendAll(border, 0);
                     sys.sendAll("*** Go in the main channel and type /join or !join to enter the tournament! ***", 0);
                     sys.sendAll(border, 0);
                     sys.sendAll("", 0);
                     return;
                 }
                 sys.sendAll("", 0);
                 sys.sendAll(border, 0);
                 sys.sendAll("*** A tournament was started by " + sys.name(src) + "! ***", 0);
                 sys.sendAll("PLAYERS: " + tournumber, 0);
                 sys.sendAll("TIER: " + tourtier, 0);
                 sys.sendAll("PRIZE: " +prize, 0);
                 sys.sendAll(border, 0);
                 sys.sendAll("*** Go in the main channel and type /join or !join to enter the tournament! ***", 0);
                 sys.sendAll(border, 0);
                 sys.sendAll("", 0);
                 return;
             }
             
             if (command == "changecount") {
                 if (tourmode != 1) {
                     sys.sendMessage(src, "Sorry, you are unable to join because the tournament has passed the sign-up phase.");
                     return;
                 }
                 var count = parseInt(commandData);
                 
                 if (isNaN(count) || count < 3) {
                     return;
                 }
                 
                 if (count < tourmembers.length) {
                     sys.sendMessage(src, "There are more than that people registered.");
                     return;
                 }
                 
                 tournumber = count;
                 
                 sys.sendAll("", 0);
                 sys.sendAll(border, 0);
                 sys.sendAll("+Bot: " +  sys.name(src) + " changed the numbers of entrants to " + count + "!", 0);
                 sys.sendAll("+Bot: Spots left: " + this.tourSpots() + ".", 0);
                 sys.sendAll(border, 0);
                 sys.sendAll("", 0);
                 
                 if (this.tourSpots() == 0 ){
                     tourmode = 2;
                     roundnumber = 0;
                     this.roundPairing();
                 }
                 
                 return;
             }
             if (command == "endtour"){
                 if (tourmode != 0){
                     tourmode = 0;
                     sys.sendAll("", 0);
                     sys.sendAll(border, 0);
                     sys.sendAll("+Bot: The tournament was cancelled by " + sys.name(src) + "!", 0);
                     sys.sendAll(border, 0);
                     sys.sendAll("", 0);
                 }else
                     sys.sendMessage(src, "Sorry, you are unable to end a tournament because one is not currently running.");
                 return;
             }
             if (command == "massmute") {
                 muteall = !muteall;
                 if (muteall)  {
                     sys.sendAll("+MuteBot: " + sys.name(src) + " muted the chat.");
                 } else {
                     sys.sendAll("+MuteBot: " + sys.name(src) + " unmuted the chat.");
                 }
                 return;
             }
	     if (command == "kick") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (sys.auth(tar) >= sys.auth(src)) {
                     sys.sendMessage(src, "+Bot: Insufficient auth to kick " + commandpart[0] + ".");
                     return;
                 }
	         if (reason == undefined && sys.name(src) == "Omega") {
	             sys.sendAll("+Bot: " + commandpart[0] + " was knocked out by " + sys.name(src) + ". Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     return;
	         }
	         if (sys.name(src) == "Omega") {
	             sys.sendAll("+Bot: " + commandpart[0] + " was knocked out by " + sys.name(src) + ". Reason: " + reason);
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     return;
                 }
	         if (reason == undefined && sys.ip(src) == "68.52.44.212") {
	             sys.sendAll("+Bot: " + sys.name(src) + " had a bread helmet! " + commandpart[0] + "'s argument was invalid! Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     return;
	         }
	         if (sys.ip(src) == "68.52.44.212") {
	             sys.sendAll("+Bot: " + sys.name(src) + " had a bread helmet! " + commandpart[0] + "'s argument was invalid! Reason: " + reason);
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " was kicked by " + sys.name(src) + ". Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " was kicked by " + sys.name(src) + ". Reason: " + reason);
	         sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                 return;
             }
             if (command == "silentkick") {
                 if (tar == undefined) {
                     return;
                 }
                 if (sys.auth(tar) >= sys.auth(src)) {
                     sys.sendMessage(src, "+Bot: Insufficient auth to silently kick " + commandData + ".");
                     return;
                 }
	         sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                 return;
             }
	     if (command == "mute") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (muted[tar]) {
                     sys.sendMessage(src, "+Bot: That player is already muted.");
                     return;
                 }
                 if (sys.auth(tar) >= sys.auth(src)) {
                     sys.sendMessage(src, "+Bot: Insufficient auth to mute " + commandpart[0] + ".");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " was muted by " + sys.name(src) + ". Reason: None given.");
	             muted[tar] = true;
	             sys.saveVal("muted_*" + sys.ip(tar), "true");
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " was muted by " + sys.name(src) + ". Reason: " + reason);
	         muted[tar] = true;
	         sys.saveVal("muted_*" + sys.ip(tar), "true");
                 return;
             }
	     if (command == "unmute") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (!muted[tar]) {
                     sys.sendMessage(src, "+Bot: That player is not muted.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " was unmuted by " + sys.name(src) + ". Reason: None given.");
	             muted[tar] = false;
	             sys.removeVal("muted_*" + sys.ip(tar));
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " was unmuted by " + sys.name(src) + ". Reason: " + reason);
	         muted[tar] = false;
	         sys.removeVal("muted_*" + sys.ip(tar));
                 return;
             }
	     if (command == "pollclose") { // Astruvis' Command
		 if (server.pollmode == 0) {
		     sys.sendMessage(src,'+Bot: Poll is already closed!'); 
		     return;}
		 server.pollmode = 0;
		 sys.sendAll('+Bot: The poll was closed by ' + sys.name(src) + '!');
		 var nu = [];
		 var tally = [];
		 for (var z in server.pollvotes) {
		     if (tally[server.pollvotes[z]] == undefined) {
			 tally[server.pollvotes[z]] = 1; }
		     else {
			 tally[server.pollvotes[z]] += 1; }
		 }
		 for (var z in tally) {
		     if (server.polloptions[z - 1] != undefined) {
			 sys.sendAll('+Bot: Votes for option #'+z+'('+ server.polloptions[z - 1] +') - ' + tally[z]); }
		 }
		 server.pollmode = 0;
		 return; }
	     if (command == "newpoll") { // Astruvis' Command
		 if (commandpart[0] == undefined || commandpart[1] == undefined) {
		     sys.sendMessage(src,'+Bot: Error.');
		     return; }
		 server.pollvotes = [];
		 server.polloptions = commandpart[1].split('/');
		 pollstuff = commandpart[0]
		 if (server.polloptions.length == 1) {
		     sys.sendMessage(src,'+Bot: Error.');
		     return; }
		 sys.sendAll('+Bot: A poll was started by ' + sys.name(src) + '!')
		 sys.sendAll('+Bot: Poll; ' + pollstuff);
		 sys.sendAll('+Bot: Please vote! Use /vote option');
		 for ( var z in server.polloptions ) {
		     sys.sendAll('+Bot: Option #'+(z*1+1)+': '+ server.polloptions[z]); }
		 server.pollmode = 1;
		 return; 
		if (command == "sendall") {
	         var namesrc = sys.name(src);
                 if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
                     sys.stopEvent();
                     sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
                     return;
                 }
	         if (muteall && sys.auth(src) == 0) {
                     sys.sendMessage(src, "+Bot: You cannot use this command while massmute is in effect. Please wait until it is over.");
	             return;
	         } else {
                     sys.sendAll(commandData, channel);
                     return;
                 }
             }
}
             /** Now for the administrator commands **/
             if (sys.auth(src) < 2) {
                 return;
             }
	     if (command == "clearbook"){
	         var get = sys.getFileContent("gb");
	         sys.deleteFile("gb");
                 sys.sendAll("The guest book was cleared!");
	         sys.writeToFile("gb","This is the guest book. Comments are listed below:<br><br>");
	         return;
	     }
	     if (command == "ban") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     //sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     sys.ban(commandpart[0]);
                     return;
                 }
                 if (sys.auth(tar) >= sys.auth(src)) {
                     sys.sendMessage(src, "+Bot: Insufficient auth to ban " + commandpart[0] + ".");
                     return;
                 }
	         if (reason == undefined && sys.ip(src) == "68.52.44.212") {
	             sys.sendAll("+Bot: " + sys.name(src) + " beat " + commandpart[0] + " with a stick until " + commandpart[0] + " spilled out candy! Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         if (sys.ip(src) == "68.52.44.212") {
	             sys.sendAll("+Bot: " + sys.name(src) + " beat " + commandpart[0] + " with a stick until " + commandpart[0] + " spilled out candy! Reason: " + reason);
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
                 }
	         if (reason == undefined && sys.name(src) == "Omega") {
	             sys.sendAll("+Bot: " + commandpart[0] + " got hit by " + sys.name(src) + "'s amazing sword-play. Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         if (sys.name(src) == "Omega") {
	             sys.sendAll("+Bot: " + commandpart[0] + " got hit by " + sys.name(src) + "'s amazing sword-play. Reason: " + reason);
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " was banned by " + sys.name(src) + ". Reason: None given.");
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " was banned by " + sys.name(src) + ". Reason: " + reason);
	         sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                 sys.ban(commandpart[0]);
                 return;
             }
	     if (command == "silentban") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     //              sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     sys.ban(commandpart[0]);
                     return;
                 }
                 if (sys.auth(tar) >= sys.auth(src)) {
                     sys.sendMessage(src, "+Bot: Insufficient auth to ban " + commandpart[0] + ".");
                     return;
                 }
	         if (reason == undefined && sys.ip(src) == "68.52.44.212") {
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         if (sys.ip(src) == "68.52.44.212") {
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
                 }
	         if (reason == undefined && sys.name(src) == "Omega") {
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         if (sys.name(src) == "Omega") {
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
                 }
	         if (reason == undefined) {
	             sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                     sys.ban(commandpart[0]);
                     return;
	         }
	         sys.setTimer("sys.kick(" + tar + ")", 1, 0);
                 sys.ban(commandpart[0]);
                 return;
             }
             if (command == "changerating") {
                 var data =  commandData.split(' -- ');
                 if (data.length != 3) {
                     sys.sendMessage(src, "+Bot: You need to give 3 parameters.");
                     return;
                 }
                 var player = data[0];
                 var tier = data[1];
                 var rating = parseInt(data[2]);
                 
                 sys.changeRating(player, tier, rating);
                 sys.sendMessage(src, "+Bot: Rating of " + player + " in tier " + tier + " was changed to " + rating);
                 return;
             }
             if (command == "showteam") {
                 var info = this.importable(tar);
                 sys.sendMessage(src, "", channel);
                 for (var x=0; x < info.length; ++x) {
                     sys.sendMessage(src, info[x], channel);
                 }
                 return;
             }
             if (command == "reset") {
                 this.serverStartUp();
                 sys.sendAll("+Server: The server script variables were reset.");
                 return;
             }
             if (command == "memorydump") {
                 sys.sendMessage(src, sys.memoryDump());
                 return;
             }
             if (command == "clearchat") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to clear the chat.");
                     return;
                 }
                 var c;
                 for (c=0;c<2999;c++) {
                     sys.sendAll("");
                 }
                 sys.clearChat();
                 return;
             }
             if (command == "resetpass") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to reset passwords.");
                     return;
                 }
                 if (sys.name(tar) == "Axel") {
     	             sys.sendMessage(src, "+Bot: The password for the server creators cannot be reset.");
		     return;
	         }
                 if (sys.name(tar) == "Saeyru") {
     	             sys.sendMessage(src, "+Bot: The password for the server creators cannot be reset.");
		     return;
	         }
	         if (sys.name(tar) == "Misora") {
     	             sys.sendMessage(src, "+Bot: The password for the server creators cannot be reset.");
		     return;
	         }
	         sys.clearPass(commandData)
	         return;
	     }
             if (command == "serverprivate") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to make the server private.");
                     return;
                 }
	         sys.makeServerPublic(false);
                 return;
             }
             if (command == "serverpublic") {
                 if (sys.name(src) != "Axel" && sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to make the server public.");
                 }
	         sys.makeServerPublic(true);
                 return;
             }
	     if (command == "user") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (sys.name(tar) == "Axel") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
                 if (sys.name(tar) == "Saeyru") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (sys.name(tar) == "Misora") {
		     sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a user by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 0);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a user by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 0);
                 return;
             }
	     if (command == "mod") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (sys.name(tar) == "Axel") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
                 if (sys.name(tar) == "Saeyru") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (sys.name(tar) == "Misora") {
		     sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a moderator by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 1);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a moderator by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 1);
                 return;
             }
	     if (command == "admin") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
                 if (sys.name(tar) == "Axel") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
                 if (sys.name(tar) == "Saeyru") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (sys.name(tar) == "Misora") {
		     sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became an administrator by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 2);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became an administrator by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 2);
                 return;
             }
	     if (command == "owner") {
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (sys.name(tar) == "Axel") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
                 if (sys.name(tar) == "Saeyru") {
     	             sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
	         if (sys.name(tar) == "Misora") {
		     sys.sendMessage(src, "+Bot: The authority level for the server creators cannot be changed.");
		     return;
	         }
                 if (tar == undefined) {
                     sys.sendMessage(src, "+Bot: Either specify a name or that player is not on.");
                     return;
                 }
	         if (reason == undefined) {
	             sys.sendAll("+Bot: " + commandpart[0] + " became a server owner by " + sys.name(src) + ". Reason: None given.");
                     sys.changeAuth(tar, 3);
                     return;
	         }
	         sys.sendAll("+Bot: " + commandpart[0] + " became a server owner by " + sys.name(src) + ". Reason: " + reason);
                 sys.changeAuth(tar, 3);
                 return;
             }
	     if (command == "unban") {
                 if (sys.name(src) != "Axel"&& sys.name(src) != "Saeyru" && sys.name(src) != "Misora") {
                     sys.sendMessage(src, "+Bot: You do not have the ability to unban a player.");
                     return;
                 }
                 commandpart = commandData.split(':');
		 var tar = sys.id(commandpart[0])
		 reason = commandpart[1]
                 if (tar == undefined) {
                     sys.unban(commandpart[0]);
                     return;
                 }
	         if (reason == undefined) {
                     sys.unban(commandpart[0]);
                     return;
	         }
                 sys.unban(commandpart[0]);
                 return;
             }
             if (command == "stopbattles") {
                 battlesStopped = !battlesStopped;
                 if (battlesStopped)  {
                     sys.sendAll("+BattleBot: Battles are now disabled.");
                 } else {
                     sys.sendAll("+BattleBot: Battles are now enabled.");
                 }
             }
             return;
         }
         if (channel == mafiachan && mafia.ticks > 0 && mafia.state!="blank" && !mafia.isInGame(sys.name(src)) && sys.auth(src) <= 0) {
             sys.stopEvent();
             sys.sendMessage(src, "±Game: You're not playing, so shush! Go in another channel to talk!", mafiachan);
             return;
         }
         if (channel != mafiachan && sys.auth(src) == 0 && muteall && message != "!join" && message != "/rules" && message != "/join" && message != "!rules" && message != "/leave" && message != "!leave" && message != "!mutecheck" && message != "/mutecheck" && message != "!massmutecheck" && message != "/massmutecheck") {
             sys.sendMessage(src, "+Bot: You cannot speak while massmute is in effect. Please wait until it is over.");
             sys.stopEvent();
             return;
         }
     }
     ,

     tourSpots : function() {
         return tournumber - tourmembers.length;
     }

     ,

     roundPairing : function() {
         roundnumber += 1;

         battlesStarted = [];
         tourbattlers = [];
         battlesLost = [];
         
         if (tourmembers.length == 1) {
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("", 0);
             sys.sendAll("The winner of the tournament is... : " + tourplayers[tourmembers[0]], 0);
             sys.sendAll("", 0);
             sys.sendAll("*** Congratulations, " + tourplayers[tourmembers[0]] + ", on your success! ***", 0);
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("", 0);
             tourmode = 0;
             return;
         }
         
         var finals = tourmembers.length == 2;
         
         if (!finals) {
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("*** Current Round of " + tourtier + " tournament: " + roundnumber + " ***", 0);
             sys.sendAll(border, 0);
         }
         else {
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("*** Current Round of " + tourtier + " tournament: " + roundnumber + " (Final) ***\n", 0);
             sys.sendAll(border, 0);
         }
         
         var i = 0;
         while (tourmembers.length >= 2) {
             i += 1;
             var x1 = sys.rand(0, tourmembers.length);
             tourbattlers.push(tourmembers[x1]);
             var name1 = tourplayers[tourmembers[x1]];
             tourmembers.splice(x1,1);
             
             
             x1 = sys.rand(0, tourmembers.length);
             tourbattlers.push(tourmembers[x1]);
             var name2 = tourplayers[tourmembers[x1]];
             tourmembers.splice(x1,1);
             
             battlesStarted.push(false);
             
             if (!finals)
                 sys.sendAll (i + "." + this.padd(name1) + " VS " + name2, 0);
             else
                 sys.sendAll ("  " + this.padd(name1) + " VS " + name2, 0);
         }
         
         if (tourmembers.length > 0) {
             sys.sendAll ("", 0);
             sys.sendAll ("*** " + tourplayers[tourmembers[0]] + " is randomly selected to go to the next round! ***", 0);
         }
         
         if (finals) {
             sys.sendAll(border, 0);
             sys.sendAll("", 0);
         } else {
             sys.sendAll(border, 0);
             sys.sendAll("", 0);
         }
     }

     ,

     padd : function(name) {
         var ret = name;
         
         while (ret.length < 20) ret = ' ' + ret;
         
         return ret;
     }

     ,

     isInTourney : function (name) {
         var name2 = name.toLowerCase();
         return name2 in tourplayers;
     }

     ,

     tourOpponent : function (nam) {
         var name = nam.toLowerCase();
         
         var x = tourbattlers.indexOf(name);
         
         if (x != -1) {
             if (x % 2 == 0) {
                 return tourbattlers[x+1];
             } else {
                 return tourbattlers[x-1];
             }
         }
         
         return "";
     }

     ,

     areOpponentsForTourBattle : function(src, dest) {
         return this.isInTourney(sys.name(src)) && this.isInTourney(sys.name(dest)) && this.tourOpponent(sys.name(src)) == sys.name(dest).toLowerCase();
     }
     ,

     areOpponentsForTourBattle2 : function(src, dest) {
         return this.isInTourney(src) && this.isInTourney(dest) && this.tourOpponent(src) == dest.toLowerCase();
     }
     ,

     ongoingTourneyBattle : function (name) {
         return tourbattlers.indexOf(name.toLowerCase()) != -1 && battlesStarted[Math.floor(tourbattlers.indexOf(name.toLowerCase())/2)] == true;
     }
     ,

     tourBattleEnd : function(src, dest)
     {
         if (!this.areOpponentsForTourBattle2(src, dest) || !this.ongoingTourneyBattle(src))
             return;
         battlesLost.push(src);
         battlesLost.push(dest);
         
         var srcL = src.toLowerCase();
         var destL = dest.toLowerCase();
         
         battlesStarted.splice(Math.floor(tourbattlers.indexOf(srcL)/2), 1);
         tourbattlers.splice(tourbattlers.indexOf(srcL), 1);
         tourbattlers.splice(tourbattlers.indexOf(destL), 1);
         tourmembers.push(srcL);
         delete tourplayers[destL];
         
         if (tourbattlers.length != 0 || tourmembers.length > 1) {
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("+Bot: " + src + " advances to the next round.", 0);
             sys.sendAll("+Bot: " + dest + " is out of the tournament.", 0);
             sys.sendAll(border, 0);
         }
         
         if (tourbattlers.length > 0) {
             sys.sendAll("", 0);
             sys.sendAll(border, 0);
             sys.sendAll("*** Battles left: " + tourbattlers.length/2 + ". ***", 0);
             sys.sendAll(border, 0);
             sys.sendAll("", 0);
             return;
         }
         
         this.roundPairing();
     }

     ,

     beforeChangeTier : function(src, oldtier, newtier) {
         this.monotypecheck(src, newtier)
         this.weatherlesstiercheck(src, newtier)
     }

     ,

     testVgcOk : function (src)
     {
         if (sys.teamPoke(src, 4) != 0 || sys.teamPoke(src, 5) != 0) {
             return false;
         }
         
         if (sys.hasTeamItem(src, sys.itemNum("Soul Dew"))) {
             return false;
         }
         
         var semiCount = 0;
         
         for (var i = 0; i < 4; i += 1) {
             if (typeof(semiUbers[sys.teamPoke(src, team, i)]) != "undefined") {
                 semiCount += 1;
             }
         }
         
         if (semiCount > 2) {
             return false;
         }
         
         return true;
     }
     ,

     eventMovesCheck : function(src)
     {
         for (var team = 0; team < sys.teamCount(src); team++) {
             for (var i = 0; i < 6; i++) {
                 var poke = sys.teamPoke(src, team, i);
                 if (poke in pokeNatures) {
                     for (x in pokeNatures[poke]) {
                         if (sys.hasTeamPokeMove(src, team, i, x) && sys.teamPokeNature(src, team, i) != pokeNatures[poke][x])
                         {
                             sys.sendMessage(src, "+CheckBot: " + sys.pokemon(poke) + " with " + sys.move(x) + " must be of the " + sys.nature(pokeNatures[poke][x]) + " nature. Change it in the team builder.");
                             sys.stopEvent();
                             sys.changePokeNum(src, team, i, 0);
                         }
                         
                     }
                 }
             }
         }
     }
     ,
     littleCupCheck : function(src, se) {
         for (var team = 0; team < sys.teamCount(src); team++) {
             if (["LC Wifi", "LC Ubers Wifi"].indexOf(sys.tier(src, team)) == -1) {
                 return; // only care about these tiers
             }
             for (var i = 0; i < 6; i++) {
                 var x = sys.teamPoke(src, team, i);
                 if (x != 0 && sys.hasDreamWorldAbility(src, team, i) && lcpokemons.indexOf(x) != -1) {
                     if (se) {
                         sys.sendMessage(src, "+CheckBot: " + sys.pokemon(x) + " is not allowed with a Dream World ability in this tier. Change it in the teambuilder.");
                     }

                     if (sys.tier(src, team) == "LC Wifi" && sys.hasLegalTeamForTier(src, team, "LC Dream World") || sys.tier(src, team) == "LC Ubers Wifi" && sys.hasLegalTeamForTier(src, team, "Dream World")) {
                         sys.changeTier(src, team, "LC Dream World");
                     }else {
                         if (se)
                             sys.changePokeNum(src, i, 0);

                     }
                     if (se)
                         sys.stopEvent();
                 }
             }
         }
     }
     ,
     dreamWorldAbilitiesCheck : function(src, se) {
         for (var team = 0; team < sys.teamCount(src); team++) {
             if (sys.gen(src, team) < 5)
                 return;

             if (["Dream World", "Dream World Ubers", "LC Dream World", "Monotype", "Dream World UU", "Dream World LU", "Weatherless", "Challenge Cup" , "1v1 Challenge Cup", "Uber Triples", "OU Triples", "Uber Doubles", "OU Doubles", "Shanai Cup", "Monocolour"].indexOf(sys.tier(src, team)) != -1) {
                 return; // don't care about these tiers
             }

             for (	var team = 0; team < sys.teamCount(src); team++) {
                 for (var i = 0; i < 6; i++) {
                     var x = sys.teamPoke(src, team, i);
                     pokemon = sys.pokemon(sys.teamPoke(src, team, i))
                     if (x != 0 && pokemon.toLowerCase() != "gliscor" && pokemon.toLowerCase() != "gligar" && pokemon.toLowerCase() != "torchic" && pokemon.toLowerCase() != "combusken" && pokemon.toLowerCase() != "blaziken" && pokemon.toLowerCase() != "carvanha" && pokemon.toLowerCase() != "sharpedo" && sys.hasDreamWorldAbility(src, team, i) && (!(x in dwpokemons) || (breedingpokemons.indexOf(x) != -1 && sys.compatibleAsDreamWorldEvent(src, team, i) != true))) {
                         if (se) {
                             if (!(x in dwpokemons))
                                 sys.sendMessage(src, "+CheckBot: " + sys.pokemon(x) + " is not allowed with a Dream World ability in this tier. Change it in the teambuilder.");
                             else
                                 sys.sendMessage(src, "+CheckBot: " + sys.pokemon(x) + " has to be Male and have no egg moves with its Dream World ability in  " + sys.tier(src, team) + " tier. Change it in the teambuilder.");
                         }
                         if (sys.tier(src, team) == "Wifi" && sys.hasLegalTeamForTier(src, team, "Dream World")) {
                             sys.changeTier(src, team, "Dream World");
                         } else if (sys.tier(src, team) == "Wifi" && sys.hasLegalTeamForTier(src, team, "Dream World Ubers")) {
                             sys.changeTier(src, team, "Dream World Ubers");
                         } else if (sys.tier(src, team) == "Wifi Ubers") {
                             sys.changeTier(src, team, "Dream World Ubers");
                         }
                         else if (sys.tier(src, team) == "1v1 Gen 5" && sys.hasLegalTeamForTier(src, team, "Dream World")) {
                             sys.changeTier(src, team, "Dream World");
                         }
                         else if (sys.tier(src, team) == "1v1 Gen 5" && sys.hasLegalTeamForTier(src, team, "Dream World Ubers")) {
                             sys.changeTier(src, team, "Dream World Ubers");
                         }
                         else if (sys.tier(src, team) == "Wifi UU" && sys.hasLegalTeamForTier(src, team, "Dream World UU")) {
                             sys.changeTier(src, team, "Dream World UU");
                         } else if (sys.tier(src, team) == "Wifi LU" && sys.hasLegalTeamForTier(src, team, "Dream World LU")) {
                             sys.changeTier(src, team, "Dream World LU");
                         }
                         else if (sys.tier(src, team) == "LC Wifi" && sys.hasLegalTeamForTier(src, team, "LC Wifi") || sys.tier(src, team) == "LC Ubers Wifi" && sys.hasLegalTeamForTier(src, team, "LC Ubers Wifi")) {
                             sys.changeTier(src, team, "LC Dream World");
                         }else {
                             if (se)
                                 sys.changePokeNum(src, team, i, 0);

                         }
                         if (se)
                             sys.stopEvent();
                     }
                 }
             }
         }
     }
     ,
     inconsistentCheck : function(src, se) {
         for (var team = 0; team < sys.teamCount(src); team++) {
             for (var i = 0; i < 6; i++) {
                 var x = sys.teamPoke(src, team, i);
                 
                 if (x != 0 && inpokemons.indexOf(x) != -1 && sys.hasDreamWorldAbility(src, team, i)) {
                     if (se)
                         sys.sendMessage(src, "+CheckBot: " + sys.pokemon(x) + " is not allowed with the Moody ability in this tier. Change it in the team builder.");
                     sys.changeTier(src, team, "Challenge Cup");
		     sys.sendMessage(src, "+CheckBot: Tier changed to Challenge Cup.");
                     if (se)
                         sys.stopEvent();
                 }
             }
         }
     }
     ,
     weatherlesstiercheck : function(src, tier) {
         for (var team = 0; team < sys.teamCount(src); team++) {
             if (!tier) tier = sys.tier(src, team);
             if (tier != "Weatherless") return;
             for (var i = 0; i < 6; i++){
                 ability = sys.ability(sys.teamPokeAbility(src, team, i))
                 if(ability.toLowerCase() == "drizzle" || ability.toLowerCase() == "drought" || ability.toLowerCase() == "snow warning" || ability.toLowerCase() == "sand stream") {
                     sys.sendMessage(src, "+Bot: Your team has a Pokemon with the ability: " + ability + ", please remove before entering this tier.");
                     if(sys.hasLegalTeamForTier(src, team, "Dream World")) {
                         if(sys.hasLegalTeamForTier(src, team, "Wifi")) {
                             sys.changeTier(src, team, "Wifi");
                             sys.stopEvent()
                             return;
                         }
                         sys.changeTier(src, team, "Dream World");
                         sys.stopEvent()
                         return;
                     }
                     if(sys.hasLegalTeamForTier(src, team, "Wifi Ubers")) {
                         sys.changeTier(src, team, "Wifi Ubers");
                         sys.stopEvent()
                         return;
                     }
                     sys.changeTier(src, team, "Dream World Ubers");
                     sys.stopEvent()
                     return;
                 }
             }
         }
     }
     ,
     monotypecheck : function(src, tier) {
         for (var team = 0; team < sys.teamCount(src); team++) {
             if (!tier) tier = sys.tier(src, team);
             if (tier != "Monotype") return; // Only interested in monotype
             TypeA = sys.pokeType1(sys.teamPoke(src, team, 0), 5)
             TypeB = sys.pokeType2(sys.teamPoke(src, team, 0), 5)
             for (var i = 1; i < 6 ; i++) {
                 temptypeA = sys.pokeType1(sys.teamPoke(src, team, i), 5)
                 temptypeB = sys.pokeType2(sys.teamPoke(src, team, i), 5)
                 if (sys.teamPoke(src, team, i) == 0){
                     temptypeA = TypeA
                 }
                 if(checkType != undefined) {
                     k=3             
                 }
                 if(i==1){
                     k=1
                 }
                 if(TypeB !=0){
                     if(temptypeA == TypeA && temptypeB == TypeB && k == 1 || temptypeA == TypeB && temptypeB == TypeA && k == 1){
                         k=2    
                     }
                 }
                 if (temptypeA == TypeA && k == 1 || temptypeB == TypeA && k == 1) {
                     var checkType=TypeA
                 }
                 if (temptypeA == TypeB && k == 1 || temptypeB == TypeB && k == 1) {
                     var checkType=TypeB
                 }
                 if(i>1 && k == 2) {
                     k=1
                     if(temptypeA == TypeA && temptypeB == TypeB && k == 1 || temptypeA == TypeB && temptypeB == TypeA && k == 1){
                         k=2
                     }
                     if (temptypeA == TypeA && k == 1 || temptypeB == TypeA && k == 1) {
                         var checkType=TypeA
                     }
                     if (temptypeA == TypeB && k == 1 || temptypeB == TypeB && k == 1) {
                         var checkType=TypeB
                     }
                 }
                 if (k==3){
                     
                     if (temptypeA != checkType && temptypeB != checkType) {
                         
                         sys.sendMessage(src, "+Bot: Your team is not eligible to battle in the Monotype tier because " + sys.pokemon(sys.teamPoke(src, team, i)) + " is not of the " + sys.type(checkType) + " type!");
                         if(sys.hasLegalTeamForTier(src, team, "Dream World")) {
                             if(sys.hasLegalTeamForTier(src, team, "Wifi")) {
                                 sys.changeTier(src, team, "Wifi");
	                         sys.sendMessage(src, "+CheckBot: Tier changed to Wifi.");
                                 sys.stopEvent()
                                 return;
                             }
                             sys.changeTier(src, team, "Dream World");
	                     sys.sendMessage(src, "+CheckBot: Tier changed to Dream World.");
                             sys.stopEvent()
                             return;
                         }
                         if(sys.hasLegalTeamForTier(src, team, "Wifi Ubers")) {
                             sys.changeTier(src, team, "Wifi Ubers");
	                     sys.sendMessage(src, "+CheckBot: Tier changed to Wifi Ubers.");
                             sys.stopEvent()
                             return;
                         }
                         sys.changeTier(src, team, "Dream World Ubers");
	                 sys.sendMessage(src, "+CheckBot: Tier changed to Dream World Ubers.");
                         sys.stopEvent()
                         return;
                     }
                 }
                 
                 if(k==1) {
                     if (temptypeA != TypeA && temptypeB != TypeA && temptypeA != TypeB && temptypeB != TypeB) {
                         sys.sendMessage(src, "+Bot: Your team is not eligible to battle in the Monotype tier because " + sys.pokemon(sys.teamPoke(src, team, i)) + " does not share the correct type with " + sys.pokemon(sys.teamPoke(src, team, 0)) + "!")
                         
                         if(sys.hasLegalTeamForTier(src, team, "Dream World")) {
                             if(sys.hasLegalTeamForTier(src, team, "Wifi")) {
                                 sys.changeTier(src, team, "Wifi");
	                         sys.sendMessage(src, "+CheckBot: Tier changed to Wifi.");
                                 sys.stopEvent()
                                 return;
                             }
                             sys.changeTier(src, team, "Dream World");
	                     sys.sendMessage(src, "+CheckBot: Tier changed to Dream World.");
                             sys.stopEvent()
                             return;
                         }
                         if(sys.hasLegalTeamForTier(src, team, "Wifi Ubers")) {
                             sys.changeTier(src, team, "Wifi Ubers");
	                     sys.sendMessage(src, "+CheckBot: Tier changed to Wifi Ubers.");
                             sys.stopEvent()
                             return;
                         }
                         sys.changeTier(src, team, "Dream World Ubers");
	                 sys.sendMessage(src, "+CheckBot: Tier changed to Dream World Ubers.");
                         sys.stopEvent()
                         return;
                     }
                     }

beforeLogIn : function(src) {
    var ip = sys.ip(src);
    // auth can evade rangebans and namebans
    if (sys.auth(src) > 0) {
        return;
    }
    var allowedIps = ["74.115.245.16","74.115.245.26"];
    if (this.isRangeBanned(ip) && allowedIps.indexOf(ip) == -1 && script.allowedRangeNames.indexOf(sys.name(src).toLowerCase()) == -1) {
        normalbot.sendMessage(src, 'You are banned!');
        sys.stopEvent();
        return;
    }
    if (proxy_ips.hasOwnProperty(ip)) {
        normalbot.sendMessage(src, 'You are banned for using proxy!');
        sys.stopEvent();
        return;

    }
    if (this.nameIsInappropriate(src)) {
        sys.stopEvent();
    }
}
                 }
             }
             
         }
     }
 })
