/**
 * @license Copyright (C) 2014 
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/> .
 */


(function(){

var kill = function(){
    clearInterval(esBot.room.autodisableInterval);
    clearInterval(esBot.room.afkInterval);
    esBot.status = false;
    console.log("FarofinhaBot morreu.");
}

var storeToStorage = function(){
    localStorage.setItem("esBotRoomSettings", JSON.stringify(esBot.roomSettings));
    localStorage.setItem("esBotRoom", JSON.stringify(esBot.room));
    var esBotStorageInfo = {
        time: Date.now(),
        stored: true,
        version: esBot.version,
    };
    localStorage.setItem("esBotStorageInfo", JSON.stringify(esBotStorageInfo));

};

var retrieveFromStorage = function(){
    var info = localStorage.getItem("esBotStorageInfo");
    if(info === null) API.chatLog("No previous data found.");
    else{
        var settings = JSON.parse(localStorage.getItem("esBotRoomSettings"));
        var room = JSON.parse(localStorage.getItem("esBotRoom"));
        var elapsed = Date.now() - JSON.parse(info).time;
        if((elapsed < 1*60*60*1000)){
            API.chatLog('Procurando dados anteriores...');
            for(var prop in settings){
                esBot.roomSettings[prop] = settings[prop];
            }
            //esBot.roomSettings = settings;
            esBot.room.users = room.users;
            esBot.room.afkList = room.afkList;
            esBot.room.historyList = room.historyList;
            esBot.room.mutedUsers = room.mutedUsers;
            esBot.room.autoskip = room.autoskip;
            esBot.room.roomstats = room.roomstats;
            esBot.room.messages = room.messages;
            esBot.room.queue = room.queue;
            API.chatLog('Dados carregados!');
        }
    }
    var json_sett = null;
    var roominfo = document.getElementById("room-info");
    var info = roominfo.innerText;
    var ref_bot = "@TBRBot=";
    var ind_ref = info.indexOf(ref_bot);
    if(ind_ref > 0){
        var link = info.substring(ind_ref + ref_bot.length, info.length);
        if(link.indexOf(" ") < link.indexOf("\n")) var ind_space = link.indexOf(" ");
        else var ind_space = link.indexOf("\n");
        link = link.substring(0,ind_space);
        $.get(link, function(json){
            if(json !== null && typeof json !== "Indefinido"){
                var json_sett = JSON.parse(json);
                for(var prop in json_sett){
                    esBot.roomSettings[prop] = json_sett[prop];
                }
            }
        });
    }

};

var esBot = {
        version: "1.1.3",        
        status: false,
        name: "TBRbot",
        creator: "Notturninho",
        loggedInID: null,
        scriptLink: "https://rawgit.com/motelbible/farofaBot/master/farofaBot.js",
        cmdLink: "https://github.com/motelbible/farofaBot/blob/master/commands.md",
        roomSettings: {
            maximumAfk: 90,
            afkRemoval: true,                
            maximumDc: 90,                                
            bouncerPlus: true,                
            lockdownEnabled: false,                
            lockGuard: false,
            maximumLocktime: 10,                
            cycleGuard: true,
            maximumCycletime: 10,                
            timeGuard: true,
            maximumSongLength: 10,                
            autodisable: true,                
            commandCooldown: 30,
            usercommandsEnabled: true,                
            lockskipPosition: 3,
            lockskipReasons: [ ["theme", "Essa música não se encaixa nos padrões. "], 
                    ["op", "Essa música é overplayed. "], 
                    ["history", "Essa música já foi tocada. "], 
                    ["mix", "Mixes não são permitidos. "], 
                    ["sound", "Má qualidade de áudio ou sem áudio. "],
                    ["nsfw", "A música que você tocou tem conteúdo NSFW. "], 
                    ["unavailable", "Música indisponível para alguns usuários. "] 
                ],
            afkpositionCheck: 15,
            afkRankCheck: "ambassador",                
            motdEnabled: false,
            motdInterval: 25,
            motd: "Por que colocar uma toalha no cesto de roupa suja, se quando você sai do chuveiro você está limpo?",
            filterChat: true,
            etaRestriction: true,
            welcome: false,
            opLink: null,
            rulesLink: "Regras básicas de qualquer sala vlwflw",
            themeLink: null,
            fbLink: "",
            youtubeLink: null,
            website: "https://www.facebook.com/groups/teambrasilplugdj/",
            intervalMessages: [],
            messageInterval: 38,
            songstats: false,                      
        },        
        room: {        
            users: [],                
            afkList: [],                
            mutedUsers: [],
            bannedUsers: [],
            skippable: true,
            usercommand: true,
            allcommand: true,   
            afkInterval: null,
            autoskip: false,
            autoskipTimer: null,
            autodisableInterval: null,
            autodisableFunc: function(){
                if(esBot.status && esBot.roomSettings.autodisable){
                    API.sendChat('!afkdisable');
                    API.sendChat('!joindisable');
                }
            },
            queueing: 0,
            queueable: true,
            currentDJID: null,
            historyList: [],
            cycleTimer: setTimeout(function(){},1),                
            roomstats: {
                    accountName: null,
                    totalWoots: 0,
                    totalCurates: 0,
                    totalMehs: 0,
                    launchTime: null,
                    songCount: 0,
                    chatmessages: 0,                
            },
            messages: {
                from: [],
                to: [],
                message: [],
            },                
            queue: {
                    id: [],
                    position: [],                             
            },
            roulette: {
                rouletteStatus: false,
                participants: [],
                countdown : null,
                startRoulette: function(){
                    esBot.room.roulette.rouletteStatus = true;
                    esBot.room.roulette.countdown = setTimeout(function(){ esBot.room.roulette.endRoulette(); }, 60 * 1000);
                    API.sendChat("/me A loteria foi iniciada. Digite !join para participar!");
                },
                endRoulette: function(){
                    esBot.room.roulette.rouletteStatus = false;
                    var ind = Math.floor(Math.random() * esBot.room.roulette.participants.length);
                    var winner = esBot.room.roulette.participants[ind];
                    esBot.room.roulette.participants = [];
                    var pos = Math.floor((Math.random() * API.getWaitList().length) + 1);
                    var user = esBot.userUtilities.lookupUser(winner);
                    var name = user.username;
                    API.sendChat("/me Temos um vencedor! @" + name + " ganhou a posição " + pos + ".");
                    setTimeout(function(winner){
                        esBot.userUtilities.moveUser(winner, pos, false);
                    }, 1*1000, winner, pos);

                },
            },
        },        
        User: function(id, name) {
            this.id = id;
            this.username = name;        
            this.jointime = Date.now();
            this.lastActivity = Date.now();         
            this.votes = {
                    woot: 0,
                    meh: 0,
                    curate: 0,
            };
            this.lastEta = null;            
            this.afkWarningCount = 0;            
            this.afkCountdown;            
            this.inRoom = true;            
            this.isMuted = false;
            this.lastDC = {
                    time: null,
                    position: null,
                    songCount: 0,
            };
            this.lastKnownPosition = null;       
        },      
        userUtilities: {
            getJointime: function(user){
                return user.jointime;
                },                        
            getUser: function(user){
                return API.getUser(user.id);
                },
            updatePosition: function(user, newPos){
                    user.lastKnownPosition = newPos;
                },                      
            updateDC: function(user){
                user.lastDC.time = Date.now();
                user.lastDC.position = user.lastKnownPosition;
                user.lastDC.songCount = esBot.room.roomstats.songCount;
                },                
            setLastActivity: function(user) {
                user.lastActivity = Date.now();
                user.afkWarningCount = 0;
                clearTimeout(user.afkCountdown);          
                },                        
            getLastActivity: function(user) {
                return user.lastActivity;
                },                        
            getWarningCount: function(user) {
                return user.afkWarningCount;
                },                        
            setWarningCount: function(user, value) {
                user.afkWarningCount = value;
                },        
            lookupUser: function(id){
                for(var i = 0; i < esBot.room.users.length; i++){
                        if(esBot.room.users[i].id === id){                                        
                                return esBot.room.users[i];
                        }
                }
                return false;
            },                
            lookupUserName: function(name){
                for(var i = 0; i < esBot.room.users.length; i++){
                        if(esBot.userUtilities.getUser(esBot.room.users[i]).username === name){
                            return esBot.room.users[i];
                        }
                }
                return false;
            },                
            voteRatio: function(id){
                var user = esBot.userUtilities.lookupUser(id);
                var votes = user.votes;
                if(votes.meh=== 0) votes.ratio = 1;
                else votes.ratio = (votes.woot / votes.meh).toFixed(2);
                return votes;
            
            },                
            getPermission: function(id){ //1 requests
                var u = API.getUser(id);
                return u.permission;
            },                
            moveUser: function(id, pos, priority){
                var user = esBot.userUtilities.lookupUser(id);
                var wlist = API.getWaitList();
                if(API.getWaitListPosition(id) === -1){                    
                    if (wlist.length < 50){
                        API.moderateAddDJ(id);
                        if (pos !== 0) setTimeout(function(id, pos){ 
                            API.moderateMoveDJ(id, pos);        
                        },1250, id, pos);
                    }                            
                    else{
                        var alreadyQueued = -1;
                        for (var i = 0; i < esBot.room.queue.id.length; i++){
                                if(esBot.room.queue.id[i] === id) alreadyQueued = i;
                        }
                        if(alreadyQueued !== -1){
                            esBot.room.queue.position[alreadyQueued] = pos;
                            return API.sendChat('/me Usuário está sendo adicionado! Posição alterada para ' + esBot.room.queue.position[alreadyQueued] + '.');
                        }
                        esBot.roomUtilities.booth.lockBooth();
                        if(priority){
                            esBot.room.queue.id.unshift(id);
                            esBot.room.queue.position.unshift(pos);
                        }
                        else{
                            esBot.room.queue.id.push(id);
                            esBot.room.queue.position.push(pos);
                        }
                        var name = user.username;
                        return API.sendChat('/me Usuário @' + name + ' adicionado a fila. Posição atual: ' + esBot.room.queue.position.length + '.');
                    }
                }
                else API.moderateMoveDJ(id, pos);                    
            },        
            dclookup: function(id){
                var user = esBot.userUtilities.lookupUser(id);                        
                if(typeof user === 'boolean') return ('/me Usuário não encontrado.');
                var name = user.username;
                if(user.lastDC.time === null) return ('/me @' + name + ' não tem se desconectado.');
                var dc = user.lastDC.time;
                var pos  = user.lastDC.position;
                if(pos === null) return ("/me A fila de espera precisa ser atualizada pelo menos uma vez para registrar posições.");
                var timeDc = Date.now() - dc;
                var validDC = false;
                if(esBot.roomSettings.maximumDc * 60 * 1000 > timeDc){
                    validDC = true;
                }                        
                var time = esBot.roomUtilities.msToStr(timeDc);
                if(!validDC) return ("/me @" + esBot.userUtilities.getUser(user).username + " desconectou-se a muito tempo atrás: " + time + ".");
                var songsPassed = esBot.room.roomstats.songCount - user.lastDC.songCount;
                var afksRemoved = 0;
                var afkList = esBot.room.afkList;
                for(var i = 0; i < afkList.length; i++){
                    var timeAfk = afkList[i][1];
                    var posAfk = afkList[i][2];
                    if(dc < timeAfk && posAfk < pos){
                            afksRemoved++;
                    }
                }
                var newPosition = user.lastDC.position - songsPassed - afksRemoved;
                if(newPosition <= 0) newPosition = 1;
                var msg = '/me @' + esBot.userUtilities.getUser(user).username + ' desconectou-se ' + time + ' atrás e deve ter posição ' + newPosition + '.';
                esBot.userUtilities.moveUser(user.id, newPosition, true);
                return msg;             
            },              
        },
        
        roomUtilities: {
            rankToNumber: function(rankString){
                var rankInt = null;
                switch (rankString){
                    case "admin":           rankInt = 10;   break;
                    case "ambassador":      rankInt = 8;    break;
                    case "host":            rankInt = 5;    break;
                    case "cohost":          rankInt = 4;    break;
                    case "manager":         rankInt = 3;    break;
                    case "bouncer":         rankInt = 2;    break;
                    case "residentdj":      rankInt = 1;    break;
                    case "user":            rankInt = 0;    break;
                }
                return rankInt;
            },        
            msToStr: function(msTime){
                var ms, msg, timeAway;
                msg = '';
                timeAway = {
                  'days': 0,
                  'hours': 0,
                  'minutes': 0,
                  'seconds': 0
                };
                ms = {
                  'day': 24 * 60 * 60 * 1000,
                  'hour': 60 * 60 * 1000,
                  'minute': 60 * 1000,
                  'second': 1000
                };                        
                if (msTime > ms.day) {
                  timeAway.days = Math.floor(msTime / ms.day);
                  msTime = msTime % ms.day;
                }
                if (msTime > ms.hour) {
                  timeAway.hours = Math.floor(msTime / ms.hour);
                  msTime = msTime % ms.hour;
                }
                if (msTime > ms.minute) {
                  timeAway.minutes = Math.floor(msTime / ms.minute);
                  msTime = msTime % ms.minute;
                }
                if (msTime > ms.second) {
                  timeAway.seconds = Math.floor(msTime / ms.second);
                }                        
                if (timeAway.days !== 0) {
                  msg += timeAway.days.toString() + 'd';
                }
                if (timeAway.hours !== 0) {
                  msg += timeAway.hours.toString() + 'h';
                }
                if (timeAway.minutes !== 0) {
                  msg += timeAway.minutes.toString() + 'm';
                }
                if (timeAway.minutes < 1 && timeAway.hours < 1 && timeAway.days < 1) {
                  msg += timeAway.seconds.toString() + 's';
                }
                if (msg !== '') {
                  return msg;
                } else {
                  return false;
                }                       
            },                
            booth:{                
                lockTimer: setTimeout(function(){},1000),                        
                locked: false,                        
                lockBooth: function(){
                    API.moderateLockWaitList(!esBot.roomUtilities.booth.locked);
                    esBot.roomUtilities.booth.locked = false;
                    if(esBot.roomSettings.lockGuard){
                        esBot.roomUtilities.booth.lockTimer = setTimeout(function (){
                            API.moderateLockWaitList(esBot.roomUtilities.booth.locked);
                        },esBot.roomSettings.maximumLocktime * 60 * 1000);
                    };                        
                },                        
                unlockBooth: function() {
                  API.moderateLockWaitList(esBot.roomUtilities.booth.locked);
                  clearTimeout(esBot.roomUtilities.booth.lockTimer);
                },                
            },                
            afkCheck: function(){
                if(!esBot.status || !esBot.roomSettings.afkRemoval) return void (0);
                    var rank = esBot.roomUtilities.rankToNumber(esBot.roomSettings.afkRankCheck);
                    var djlist = API.getWaitList();
                    var lastPos = Math.min(djlist.length , esBot.roomSettings.afkpositionCheck);
                    if(lastPos - 1 > djlist.length) return void (0);
                    for(var i = 0; i < lastPos; i++){
                        if(typeof djlist[i] !== 'undefined'){
                            var id = djlist[i].id;
                            var user = esBot.userUtilities.lookupUser(id);
                            if(typeof user !== 'boolean'){
                                var plugUser = esBot.userUtilities.getUser(user);
                                if(rank !== null && plugUser.permission <= rank){
                                    var name = plugUser.username;
                                    var lastActive = esBot.userUtilities.getLastActivity(user);
                                    var inactivity = Date.now() - lastActive;
                                    var time = esBot.roomUtilities.msToStr(inactivity);
                                    var warncount = user.afkWarningCount;
                                    if (inactivity > esBot.roomSettings.maximumAfk * 60 * 1000 ){
                                        if(warncount === 0){
                                            API.sendChat('@' + name + ', você esteve afk por ' + time + ', por favor, responda dentro de 2 minutos ou será removido.');
                                            user.afkWarningCount = 3;
                                            user.afkCountdown = setTimeout(function(userToChange){
                                                userToChange.afkWarningCount = 1; 
                                            }, 90 * 1000, user);
                                        }
                                        else if(warncount === 1){
                                            API.sendChat("@" + name + ", você será removido se não responder a tempo. [AFK]");
                                            user.afkWarningCount = 3;
                                            user.afkCountdown = setTimeout(function(userToChange){
                                                userToChange.afkWarningCount = 2;
                                            }, 30 * 1000, user);
                                        }
                                        else if(warncount === 2){
                                            var pos = API.getWaitListPosition(id);
                                            if(pos !== -1){
                                                pos++;
                                                esBot.room.afkList.push([id, Date.now(), pos]);
                                                user.lastDC = {
                                                    time: null,
                                                    position: null,
                                                    songCount: 0,
                                                };
                                                API.moderateRemoveDJ(id);
                                                API.sendChat('@' + name + ', você foi removido por estar afk por ' + time + '. Você estava na posição ' + pos + '. ');
                                            }
                                            user.afkWarningCount = 0;
                                        };
                                    }
                                }
                            }
                        }
                    }                
            },                
            changeDJCycle: function(){
                var toggle = $(".cycle-toggle");
                if(toggle.hasClass("disabled")) {
                    toggle.click();
                    if(esBot.roomSettings.cycleGuard){
                    esBot.room.cycleTimer = setTimeout(function(){
                            if(toggle.hasClass("enabled")) toggle.click();
                            }, esBot.roomSettings.cycleMaxTime * 60 * 1000);
                    }        
                }
                else {
                    toggle.click();
                    clearTimeout(esBot.room.cycleTimer);
                }        
            },
            intervalMessage: function(){
                var interval;
                if(esBot.roomSettings.motdEnabled) interval = esBot.roomSettings.motdInterval;
                else interval = esBot.roomSettings.messageInterval;
                if((esBot.room.roomstats.songCount % interval) === 0 && esBot.status){
                    var msg;
                    if(esBot.roomSettings.motdEnabled){
                        msg = esBot.roomSettings.motd;
                    }
                    else{
                        if(esBot.roomSettings.intervalMessages.length === 0) return void (0);
                        var messageNumber = esBot.room.roomstats.songCount % esBot.roomSettings.intervalMessages.length;
                        msg = esBot.roomSettings.intervalMessages[messageNumber];
                    };                              
                    API.sendChat('/me ' + msg);
                }
            },      
        },        
        eventChat: function(chat){
            for(var i = 0; i < esBot.room.users.length;i++){
                if(esBot.room.users[i].id === chat.fromID){
                        esBot.userUtilities.setLastActivity(esBot.room.users[i]);
                        if(esBot.room.users[i].username !== chat.from){
                                esBot.room.users[i].username = chat.from;
                        }
                }                            
            }                        
            if(esBot.chatUtilities.chatFilter(chat)) return void (0);
            if( !esBot.chatUtilities.commandCheck(chat) ) 
                    esBot.chatUtilities.action(chat);             
        },        
        eventUserjoin: function(user){
            var known = false;
            var index = null;
            for(var i = 0; i < esBot.room.users.length;i++){
                if(esBot.room.users[i].id === user.id){
                        known = true;
                        index = i;
                }
            }
            var greet = true;
            if(known){
                esBot.room.users[index].inRoom = true;
                var u = esBot.userUtilities.lookupUser(user.id);
                var jt = u.jointime;
                var t = Date.now() - jt;
                if(t < 10*1000) greet = false;
                else var welcome = "Bem vindo de volta, ";
            }
            else{
                esBot.room.users.push(new esBot.User(user.id, user.username));
                var welcome = "Bem vindo ";
            }    
            for(var j = 0; j < esBot.room.users.length;j++){
                if(esBot.userUtilities.getUser(esBot.room.users[j]).id === user.id){
                    esBot.userUtilities.setLastActivity(esBot.room.users[j]);
                    esBot.room.users[j].jointime = Date.now();
                }
            
            }
            if(esBot.roomSettings.welcome && greet){
                setTimeout(function(){
                    API.sendChat("" + welcome + "@" + user.username + ".");
                }, 1*1000);
            }               
        },        
        eventUserleave: function(user){
            for(var i = 0; i < esBot.room.users.length;i++){
                if(esBot.room.users[i].id === user.id){
                        esBot.userUtilities.updateDC(esBot.room.users[i]);
                        esBot.room.users[i].inRoom = false;
                }
            }
        },        
        eventVoteupdate: function(obj){
            for(var i = 0; i < esBot.room.users.length;i++){
                if(esBot.room.users[i].id === obj.user.id){
                    if(obj.vote === 1){
                        esBot.room.users[i].votes.woot++;
                    }
                    else{
                        esBot.room.users[i].votes.meh++;                                        
                    }
                }
            }               
        },        
        eventCurateupdate: function(obj){
            for(var i = 0; i < esBot.room.users.length;i++){
                if(esBot.room.users[i].id === obj.user.id){
                    esBot.room.users[i].votes.curate++;
                }
            }       
        },        
        eventDjadvance: function(obj){                
            var lastplay = obj.lastPlay;
            if(typeof lastplay === 'undefined') return void (0);
            if(esBot.roomSettings.songstats) API.sendChat("/me " + lastplay.media._previousAttributes.author + " - " + lastplay.media._previousAttributes.title + ": " + lastplay.score.positive + "W/" + lastplay.score.curates + "G/" + lastplay.score.negative + "M.")
            esBot.room.roomstats.totalWoots += lastplay.score.positive;
            esBot.room.roomstats.totalMehs += lastplay.score.negative;
            esBot.room.roomstats.totalCurates += lastplay.score.curates;
            esBot.room.roomstats.songCount++;
            esBot.roomUtilities.intervalMessage();
            esBot.room.currentDJID = API.getDJ().id;
            var alreadyPlayed = false;
            for(var i = 0; i < esBot.room.historyList.length; i++){
                if(esBot.room.historyList[i][0] === obj.media.cid){
                    var firstPlayed = esBot.room.historyList[i][1];
                    var plays = esBot.room.historyList[i].length - 1;
                    var lastPlayed = esBot.room.historyList[i][plays];
                    var now = +new Date();
                    var interfix = '';
                    if(plays > 1) interfix = 's'
                    API.sendChat('/me :repeat: Essa música já foi tocada ' + plays + ' vezes' + interfix + ' ');

                    esBot.room.historyList[i].push(+new Date());
                    alreadyPlayed = true;
                }
            }
            if(!alreadyPlayed){
                esBot.room.historyList.push([obj.media.cid, +new Date()]);
            }
            esBot.room.historyList;
            var newMedia = obj.media;
            if(esBot.roomSettings.timeGuard && newMedia.duration > esBot.roomSettings.maximumSongLength*60  && !esBot.room.roomevent){
                var name = obj.dj.username;
                API.sendChat('/me @' + name + ', sua música tem mais que ' + esBot.roomSettings.maximumSongLength + ' minutos. Peça permissão parar tocar.');
                API.moderateForceSkip();
            }
            var user = esBot.userUtilities.lookupUser(obj.dj.id);
            if(user.ownSong){
                API.sendChat('/me :up: @' + user.username + ' tem permissão para tocar sua própria música!');
                user.ownSong = false;
            }
            user.lastDC.position = null;
            clearTimeout(esBot.room.autoskipTimer);
            if(esBot.room.autoskip){
                var remaining = media.duration * 1000; 
                esBot.room.autoskipTimer = setTimeout(function(){ API.moderateForceSkip(); }, remaining - 500);
            }
            storeToStorage();

        },
        eventWaitlistupdate: function(users){
            if(users.length < 50){
                if(esBot.room.queue.id.length > 0 && esBot.room.queueable){
                    esBot.room.queueable = false;
                    setTimeout(function(){esBot.room.queueable = true;}, 500);
                    esBot.room.queueing++;
                    var id, pos;
                    setTimeout(
                        function(){
                            id = esBot.room.queue.id.splice(0,1)[0];
                            pos = esBot.room.queue.position.splice(0,1)[0];
                            API.moderateAddDJ(id,pos);
                            setTimeout(
                                function(id, pos){
                                API.moderateMoveDJ(id, pos);
                                esBot.room.queueing--;
                                if(esBot.room.queue.id.length === 0) setTimeout(function(){
                                    esBot.roomUtilities.booth.unlockBooth();
                                },1000);
                            },1000,id,pos);
                    },1000 + esBot.room.queueing * 2500);
                }
            }            
            for(var i = 0; i < users.length; i++){
                var user = esBot.userUtilities.lookupUser(users[i].id)
                esBot.userUtilities.updatePosition(user, users[i].wlIndex + 1);
            }
        },
        chatcleaner: function(chat){
            if(!esBot.roomSettings.filterChat) return false;
            if(esBot.userUtilities.getPermission(chat.fromID) > 1) return false;
            var msg = chat.message;
            var containsLetters = false;
            for(var i = 0; i < msg.length; i++){
                ch = msg.charAt(i);
                if((ch >= 'a' && ch <= 'z') || (ch >= 'A' && ch <= 'Z') || (ch >= '0' && ch <= '9') || ch === ':' || ch === '^') containsLetters = true;
            }
            if(msg === ''){
                return true;
            }
            if(!containsLetters && (msg.length === 1 || msg.length > 3)) return true;
            msg = msg.replace(/[ ,;.:\/=~+%^*\-\\"'&@#]/g,'');
            var capitals = 0;
            var ch;
            for(var i = 0; i < msg.length; i++){
                ch = msg.charAt(i);
                if(ch >= 'A' && ch <= 'Z') capitals++;
            }
            if(capitals >= 40){
                API.sendChat("/me @" + chat.from + ", desligue o capslock, por favor.");
                return true;
            }
            msg = msg.toLowerCase();
            if(msg === 'skip'){
                    API.sendChat("/me @" + chat.from + ", não peça por skips.");
                    return true;
                    }
            for (var j = 0; j < esBot.chatUtilities.spam.length; j++){
                if(msg === esBot.chatUtilities.spam[j]){
                    API.sendChat("/me @" + chat.from + ", sem spam, por favor.");
                    return true;
                    }
                }
            for (var i = 0; i < esBot.chatUtilities.beggarSentences.length; i++){
                if(msg.indexOf(esBot.chatUtilities.beggarSentences[i]) >= 0){
                    API.sendChat("/me @" + chat.from + ", não faça spam.");
                    return true;
                }
            } 
            return false;
        },        
        chatUtilities: {        
            chatFilter: function(chat){
                var msg = chat.message;
                var perm = esBot.userUtilities.getPermission(chat.fromID);
                var user = esBot.userUtilities.lookupUser(chat.fromID);
                var isMuted = false;
                for(var i = 0; i < esBot.room.mutedUsers.length; i++){
                                if(esBot.room.mutedUsers[i] === chat.fromID) isMuted = true;
                        }
                if(isMuted){
                    API.moderateDeleteChat(chat.chatID);
                    return true;
                    };
                if(esBot.roomSettings.lockdownEnabled){
                                if(perm === 0){    
                                        API.moderateDeleteChat(chat.chatID);
                                        return true;
                                }
                        };
                if(esBot.chatcleaner(chat)){
                    API.moderateDeleteChat(chat.chatID);
                    return true;
                }
                var plugRoomLinkPatt, sender;
                    plugRoomLinkPatt = /(\bhttps?:\/\/(www.)?plug\.dj[-A-Z0-9+&@#\/%?=~_|!:,.;]*[-A-Z0-9+&@#\/%=~_|])/ig;
                    if (plugRoomLinkPatt.exec(msg)) {
                      sender = API.getUser(chat.fromID);
                      if (perm === 0) {                                                              
                              API.sendChat("/me @" + chat.from + ", não divulgue outras salas aqui.");
                              API.moderateDeleteChat(chat.chatID);
                              return true;
                      }
                    }
                if(msg.indexOf('http://adf.ly/') > -1){
                    API.moderateDeleteChat(chat.chatID);
                    API.sendChat('/me @' + chat.from + ', use outro autowoot script. Sugiro PlugCubed: http://plugcubed.net/');
                    return true;
                }                    
                if(msg.indexOf('autojoin was not enabled') > 0 || msg.indexOf('AFK message was not enabled') > 0 || msg.indexOf('!afkdisable') > 0 || msg.indexOf('!joindisable') > 0 || msg.indexOf('autojoin disabled') > 0 || msg.indexOf('AFK message disabled') > 0){ 
                    API.moderateDeleteChat(chat.chatID);
                    return true;
                }                       
            return false;                        
            },                        
            commandCheck: function(chat){
                var cmd;
                if(chat.message.charAt(0) === '!'){
                        var space = chat.message.indexOf(' ');
                        if(space === -1){
                                cmd = chat.message;
                        }
                        else cmd = chat.message.substring(0,space);
                }
                else return false;
                API.moderateDeleteChat(chat.chatID); 
                var userPerm = esBot.userUtilities.getPermission(chat.fromID);
                if(chat.message !== "!join" && chat.message !== "!leave"){                            
                    if(userPerm === 0 && !esBot.room.usercommand) return void (0);
                    if(!esBot.room.allcommand) return void (0);
                }                            
                if(chat.message === '!eta' && esBot.roomSettings.etaRestriction){
                    if(userPerm < 2){
                        var u = esBot.userUtilities.lookupUser(chat.fromID);
                        if(u.lastEta !== null && (Date.now() - u.lastEta) < 1*60*60*1000){
                            API.moderateDeleteChat(chat.chatID);
                            return void (0);
                        }
                        else u.lastEta = Date.now();
                    }
                }                            
                var executed = false;                            
                switch(cmd){
                    case '!active':             esBot.commands.activeCommand.functionality(chat, '!active');                        executed = true; break;
                    case '!add':                esBot.commands.addCommand.functionality(chat, '!add');                              executed = true; break;
                    case '!afklimit':           esBot.commands.afklimitCommand.functionality(chat, '!afklimit');                    executed = true; break;
                    case '!afkremoval':         esBot.commands.afkremovalCommand.functionality(chat, '!afkremoval');                executed = true; break;
                    case '!afkreset':           esBot.commands.afkresetCommand.functionality(chat, '!afkreset');                    executed = true; break;
                    case '!afktime':            esBot.commands.afktimeCommand.functionality(chat, '!afktime');                      executed = true; break;
                    case '!autoskip':           esBot.commands.autoskipCommand.functionality(chat, '!autoskip');                    executed = true; break;
                    case '!autowoot':           esBot.commands.autowootCommand.functionality(chat, '!autowoot');                    executed = true; break;
                    case '!ba':                 esBot.commands.baCommand.functionality(chat, '!ba');                                executed = true; break;
                    case '!ban':                esBot.commands.banCommand.functionality(chat, '!ban');                              executed = true; break;
                    //case '!bouncer+':           esBot.commands.bouncerPlusCommand.functionality(chat, '!bouncer+');                 executed = true; break;
                    case '!clearchat':          esBot.commands.clearchatCommand.functionality(chat, '!clearchat');                  executed = true; break;
                    case '!commands':           esBot.commands.commandsCommand.functionality(chat, '!commands');                    executed = true; break;
                    case '!cookie':             esBot.commands.cookieCommand.functionality(chat, '!cookie');                        executed = true; break;
                    case '!cycle':              esBot.commands.cycleCommand.functionality(chat, '!cycle');                          executed = true; break;
                    //case '!cycleguard':         esBot.commands.cycleguardCommand.functionality(chat, '!cycleguard');                executed = true; break;
                    //case '!cycletimer':         esBot.commands.cycletimerCommand.functionality(chat, '!cycletimer');                executed = true; break;
                    case '!dclookup':           esBot.commands.dclookupCommand.functionality(chat, '!dclookup');                    executed = true; break;
                    case '!dc':                 esBot.commands.dclookupCommand.functionality(chat, '!dc');                          executed = true; break;
                    case '!emoji':              esBot.commands.emojiCommand.functionality(chat, '!emoji');                          executed = true; break;
                    //case '!english':            esBot.commands.englishCommand.functionality(chat, '!english');                      executed = true; break;
                    case '!eta':                esBot.commands.etaCommand.functionality(chat, '!eta');                              executed = true; break;
                    case '!fb':                 esBot.commands.fbCommand.functionality(chat, '!fb');                                executed = true; break;
                    case '!filter':             esBot.commands.filterCommand.functionality(chat, '!filter');                        executed = true; break;
                    case '!join':               esBot.commands.joinCommand.functionality(chat, '!join');                            executed = true; break;
                    case '!jointime':           esBot.commands.jointimeCommand.functionality(chat, '!jointime');                    executed = true; break;
                    case '!kick':               esBot.commands.kickCommand.functionality(chat, '!kick');                            executed = true; break;
                    case '!kill':               esBot.commands.killCommand.functionality(chat, '!kill');                            executed = true; break;
                    case '!leave':              esBot.commands.leaveCommand.functionality(chat, '!leave');                          executed = true; break;
                    case '!link':               esBot.commands.linkCommand.functionality(chat, '!link');                            executed = true; break;
                    case '!lock':               esBot.commands.lockCommand.functionality(chat, '!lock');                            executed = true; break;
                    case '!lockdown':           esBot.commands.lockdownCommand.functionality(chat, '!lockdown');                    executed = true; break;
                    case '!lockguard':          esBot.commands.lockguardCommand.functionality(chat, '!lockguard');                  executed = true; break;
                    case '!lockskip':           esBot.commands.lockskipCommand.functionality(chat, '!lockskip');                    executed = true; break;
                    case '!lockskippos':        esBot.commands.lockskipposCommand.functionality(chat, '!lockskippos');              executed = true; break;
                    //case '!locktimer':          esBot.commands.locktimerCommand.functionality(chat, '!locktimer');                  executed = true; break;
                    case '!maxlength':          esBot.commands.maxlengthCommand.functionality(chat, '!maxlength');                  executed = true; break;
                    //case '!motd':               esBot.commands.motdCommand.functionality(chat, '!motd');                            executed = true; break;
                    case '!move':               esBot.commands.moveCommand.functionality(chat, '!move');                            executed = true; break;
                    case '!mute':               esBot.commands.muteCommand.functionality(chat, '!mute');                            executed = true; break;
                    //case '!op':                 esBot.commands.opCommand.functionality(chat, '!op');                                executed = true; break;
                    case '!ping':               esBot.commands.pingCommand.functionality(chat, '!ping');                            executed = true; break;
                    case '!reload':             esBot.commands.reloadCommand.functionality(chat, '!reload');                        executed = true; break;
                    case '!remove':             esBot.commands.removeCommand.functionality(chat, '!remove');                        executed = true; break;
                    //case '!refresh':            esBot.commands.refreshCommand.functionality(chat, '!refresh');                      executed = true; break;
                    case '!restricteta':        esBot.commands.restrictetaCommand.functionality(chat, '!restricteta');              executed = true; break;
                    case '!roulette':           esBot.commands.rouletteCommand.functionality(chat, '!roulette');                    executed = true; break;
                    case '!rules':              esBot.commands.rulesCommand.functionality(chat, '!rules');                          executed = true; break;
                    //case '!sessionstats':       esBot.commands.sessionstatsCommand.functionality(chat, '!sessionstats');            executed = true; break;
                    case '!skip':               esBot.commands.skipCommand.functionality(chat, '!skip');                            executed = true; break;
                    case '!source':             esBot.commands.sourceCommand.functionality(chat, '!source');                        executed = true; break;
                    case '!status':             esBot.commands.statusCommand.functionality(chat, '!status');                        executed = true; break;
                    case '!sambar':             esBot.commands.sambarCommand.functionality(chat, '!sambar');                        executed = true; break;
                    //case '!theme':              esBot.commands.themeCommand.functionality(chat, '!theme');                          executed = true; break;
                    //case '!timeguard':          esBot.commands.timeguardCommand.functionality(chat, '!timeguard');                  executed = true; break;
                    case '!togglemotd':         esBot.commands.togglemotdCommand.functionality(chat, '!togglemotd');                executed = true; break;
                    case '!unban':              esBot.commands.unbanCommand.functionality(chat, '!unban');                          executed = true; break;
                    case '!unlock':             esBot.commands.unlockCommand.functionality(chat, '!unlock');                        executed = true; break;
                    case '!unmute':             esBot.commands.unmuteCommand.functionality(chat, '!unmute');                        executed = true; break;
                    case '!usercmdcd':          esBot.commands.usercmdcdCommand.functionality(chat, '!usercmdcd');                  executed = true; break;
                    case '!usercommands':       esBot.commands.usercommandsCommand.functionality(chat, '!usercommands');            executed = true; break;
                    //case '!voteratio':          esBot.commands.voteratioCommand.functionality(chat, '!voteratio');                  executed = true; break;
                    case '!welcome':            esBot.commands.welcomeCommand.functionality(chat, '!welcome');                      executed = true; break;
                    case '!website':            esBot.commands.websiteCommand.functionality(chat, '!website');                      executed = true; break;
                    //case '!youtube':            esBot.commands.youtubeCommand.functionality(chat, '!youtube');                      executed = true; break;
                    //case '!command': esBot.commands.commandCommand.functionality(chat, '!command'); executed = true; break;
                }
                if(executed && userPerm === 0){
                    esBot.room.usercommand = false;
                    setTimeout(function(){ esBot.room.usercommand = true; }, esBot.roomSettings.commandCooldown * 1000);                               
                }
                if(executed){
                    API.moderateDeleteChat(chat.chatID);
                    esBot.room.allcommand = false;
                    setTimeout(function(){ esBot.room.allcommand = true; }, 5 * 1000);
                }
                return executed;                                
            },                        
            action: function(chat){
                var user = esBot.userUtilities.lookupUser(chat.fromID);                        
                if (chat.type === 'message') {
                    for(var j = 0; j < esBot.room.users.length;j++){
                        if(esBot.userUtilities.getUser(esBot.room.users[j]).id === chat.fromID){
                            esBot.userUtilities.setLastActivity(esBot.room.users[j]);
                        }
                    
                    }
                }
                esBot.room.roomstats.chatmessages++;                                
            },
            spam: [
                'fuckadmins','modafuka','modafoka','mudafuka','mudafoka','caralho','buceta','vai se foder','foda-se'
            ],
            curses: [
                'nigger', 'faggot', 'nigga', 'niqqa','motherfucker','modafocka'
            ],                        
            beggarSentences: ['fanme','funme','becomemyfan','trocofa','fanforfan','fan4fan','fan4fan','hazcanfanz','fun4fun','fun4fun',
                'meufa','fanz','isnowyourfan','reciprocate','fansme','givefan','fanplz','fanpls','plsfan','plzfan','becomefan','tradefan',
                'fanifan','bemyfan','retribui','gimmefan','fansatfan','fansplz','fanspls','ifansback','fanforfan','addmefan','retribuo',
                'fantome','becomeafan','fan-to-fan','fantofan','canihavefan','pleasefan','addmeinfan','iwantfan','fanplease','ineedfan',
                'ineedafan','iwantafan','bymyfan','fannme','returnfan','bymyfan','givemeafan','sejameufa','sejameusfa','sejameuf��',
                'sejameusf��','f��please','f��pls','f��plz','fanxfan','addmetofan','fanzafan','fanzefan','becomeinfan','backfan',
                'viremmeuseguidor','viremmeuseguir','fanisfan','funforfun','anyfanaccept','anyfanme','fan4fan','fan4fan','turnmyfan',
                'turnifan','beafanofme','comemyfan','plzzfan','plssfan','procurofan','comebackafan','fanyfan','givemefan','fan=fan',
                'fan=fan','fan+fan','fan+fan','fanorfan','beacomeafanofme','beacomemyfan','bcomeafanofme','bcomemyfan','fanstofan',
                'bemefan','trocarfan','fanforme','fansforme','allforfan','fansintofans','fanintofan','f(a)nme','prestomyfan',
                'presstomyfan','fanpleace','fanspleace','givemyafan','addfan','addsmetofan','f4f','canihasfan','canihavefan',
                'givetomeafan','givemyfan','phanme','fanforafan','fanvsfan','fanturniturn','fanturninturn','sejammeufa',
                'sejammeusfa','befanofme','faninfan','addtofan','fanthisaccount','fanmyaccount','fanback','addmeforfan',
                'fans4fan','fans4fan','fanme','fanmyaccount','fanback','addmeforfan','fans4fan','fans4fan','fanme','turnfanwhocontribute',
                "bemefan","bemyfan","beacomeafanofme","beacomemyfan","becameyafan","becomeafan",
                "becomefan","becomeinfan","becomemyfan","becomemyfans","bouncerplease","bouncerpls",
                "brbrbrbr","brbrbrbr","bymyfan","canihasfan","canihavefan","caralho",
                "clickmynametobecomeafan","comebackafan","comemyfan","dosfanos","everyonefan",
                "everyonefans","exchangefan","f4f","f&n","f(a)nme","f@nme","��@nme","f4f","f4n4f4n",
                "f4nforf4n","f4nme","f4n4f4n","f��","f��","f��������please","f��������pls","f��������plz","fan:four:fan",
                'fanme','funme','becomemyfan','trocofa','fanforfan','fan4fan','fan4fan','hazcanfanz',
                'fun4fun','fun4fun','meufa','fanz','isnowyourfan','reciprocate','fansme','givefan',
                'fanplz','fanpls','plsfan','plzfan','becomefan','tradefan','fanifan','bemyfan',
                'retribui','gimmefan','fansatfan','fansplz','fanspls','ifansback','fanforfan',
                'addmefan','retribuo','fantome','becomeafan','fan-to-fan','fantofan',
                'canihavefan','pleasefan','addmeinfan','iwantfan','fanplease','ineedfan',
                'ineedafan','iwantafan','bymyfan','fannme','returnfan','bymyfan','givemeafan',
                'sejameufa','sejameusfa','sejameufã','sejameusfã','fãplease','fãpls','fãplz',
                'fanxfan','addmetofan','fanzafan','fanzefan','becomeinfan','backfan',
                'viremmeuseguidor','viremmeuseguir','fanisfan','funforfun','anyfanaccept',
                'anyfanme','fan4fan','fan4fan','turnmyfan','turnifan','beafanofme','comemyfan',
                'plzzfan','plssfan','procurofan','comebackafan','fanyfan','givemefan','fan=fan',
                'fan=fan','fan+fan','fan+fan','fanorfan','beacomeafanofme','beacomemyfan',
                'bcomeafanofme','bcomemyfan','fanstofan','bemefan','trocarfan','fanforme',
                'fansforme','allforfan','fnme','fnforfn','fansintofans','fanintofan','f(a)nme','prestomyfan',
                'presstomyfan','fanpleace','fanspleace','givemyafan','addfan','addsmetofan',
                'f4f','canihasfan','canihavefan','givetomeafan','givemyfan','phanme','but i need please fan',
                'fanforafan','fanvsfan','fanturniturn','fanturninturn','sejammeufa',
                'sejammeusfa','befanofme','faninfan','addtofan','fanthisaccount',
                'fanmyaccount','fanback','addmeforfan','fans4fan','fans4fan','fanme','bemyfanpls','befanpls','f4f','fanyfan',
                'caralho','buceta','vai se foder','foda-se'
            ],
        },
        connectAPI: function(){
            this.proxy = {
                    eventChat:                                      $.proxy(this.eventChat,                                     this),
                    eventUserskip:                                  $.proxy(this.eventUserskip,                                 this),
                    eventUserjoin:                                  $.proxy(this.eventUserjoin,                                 this),
                    eventUserleave:                                 $.proxy(this.eventUserleave,                                this),
                    eventUserfan:                                   $.proxy(this.eventUserfan,                                  this),
                    eventFriendjoin:                                $.proxy(this.eventFriendjoin,                               this),
                    eventFanjoin:                                   $.proxy(this.eventFanjoin,                                  this),
                    eventVoteupdate:                                $.proxy(this.eventVoteupdate,                               this),
                    eventCurateupdate:                              $.proxy(this.eventCurateupdate,                             this),
                    eventRoomscoreupdate:                           $.proxy(this.eventRoomscoreupdate,                          this),
                    eventDjadvance:                                 $.proxy(this.eventDjadvance,                                this),
                    eventDjupdate:                                  $.proxy(this.eventDjupdate,                                 this),
                    eventWaitlistupdate:                            $.proxy(this.eventWaitlistupdate,                           this),
                    eventVoteskip:                                  $.proxy(this.eventVoteskip,                                 this),
                    eventModskip:                                   $.proxy(this.eventModskip,                                  this),
                    eventChatcommand:                               $.proxy(this.eventChatcommand,                              this),
                    eventHistoryupdate:                             $.proxy(this.eventHistoryupdate,                            this),

            };            
            API.on(API.CHAT,                                        this.proxy.eventChat);
            API.on(API.USER_SKIP,                                   this.proxy.eventUserskip);
            API.on(API.USER_JOIN,                                   this.proxy.eventUserjoin);
            API.on(API.USER_LEAVE,                                  this.proxy.eventUserleave);
            API.on(API.USER_FAN,                                    this.proxy.eventUserfan);
            API.on(API.FRIEND_JOIN,                                 this.proxy.eventFriendjoin);
            API.on(API.FAN_JOIN,                                    this.proxy.eventFanjoin);
            API.on(API.VOTE_UPDATE,                                 this.proxy.eventVoteupdate);
            API.on(API.CURATE_UPDATE,                               this.proxy.eventCurateupdate);
            API.on(API.ROOM_SCORE_UPDATE,                           this.proxy.eventRoomscoreupdate);
            API.on(API.DJ_ADVANCE,                                  this.proxy.eventDjadvance);
            API.on(API.DJ_UPDATE,                                   this.proxy.eventDjupdate);
            API.on(API.WAIT_LIST_UPDATE,                            this.proxy.eventWaitlistupdate);
            API.on(API.VOTE_SKIP,                                   this.proxy.eventVoteskip);
            API.on(API.MOD_SKIP,                                    this.proxy.eventModskip);
            API.on(API.CHAT_COMMAND,                                this.proxy.eventChatcommand);
            API.on(API.HISTORY_UPDATE,                              this.proxy.eventHistoryupdate);
        },
        disconnectAPI:function(){                        
            API.off(API.CHAT,                                        this.proxy.eventChat);
            API.off(API.USER_SKIP,                                   this.proxy.eventUserskip);
            API.off(API.USER_JOIN,                                   this.proxy.eventUserjoin);
            API.off(API.USER_LEAVE,                                  this.proxy.eventUserleave);
            API.off(API.USER_FAN,                                    this.proxy.eventUserfan);
            API.off(API.FRIEND_JOIN,                                 this.proxy.eventFriendjoin);
            API.off(API.FAN_JOIN,                                    this.proxy.eventFanjoin);
            API.off(API.VOTE_UPDATE,                                 this.proxy.eventVoteupdate);
            API.off(API.CURATE_UPDATE,                               this.proxy.eventCurateupdate);
            API.off(API.ROOM_SCORE_UPDATE,                           this.proxy.eventRoomscoreupdate);
            API.off(API.DJ_ADVANCE,                                  this.proxy.eventDjadvance);
            API.off(API.DJ_UPDATE,                                   this.proxy.eventDjupdate);
            API.off(API.WAIT_LIST_UPDATE,                            this.proxy.eventWaitlistupdate);
            API.off(API.VOTE_SKIP,                                   this.proxy.eventVoteskip);
            API.off(API.MOD_SKIP,                                    this.proxy.eventModskip);
            API.off(API.CHAT_COMMAND,                                this.proxy.eventChatcommand);
            API.off(API.HISTORY_UPDATE,                              this.proxy.eventHistoryupdate);
        },
        startup: function(){
            var u = API.getUser();
            if(u.permission < 2) return API.chatLog("Somente seguranças ou mais podem usar chatbot.");
            if(u.permission === 2) return API.chatLog("O bot não pode mover enquanto for bouncer/segurança.");
            this.connectAPI();
            retrieveFromStorage();
            if(esBot.room.roomstats.launchTime === null){
                esBot.room.roomstats.launchTime = Date.now();
            }
            for(var j = 0; j < esBot.room.users.length; j++){
                esBot.room.users[j].inRoom = false;
            }                        
            var userlist = API.getUsers();
            for(var i = 0; i < userlist.length;i++){
                var known = false;
                var ind = null;
                for(var j = 0; j < esBot.room.users.length; j++){
                    if(esBot.room.users[j].id === userlist[i].id){
                        known = true;
                        ind = j;
                    }
                }
                if(known){
                        esBot.room.users[ind].inRoom = true;
                }
                else{
                        esBot.room.users.push(new esBot.User(userlist[i].id, userlist[i].username));
                        ind = esBot.room.users.length - 1;
                }
                var wlIndex = API.getWaitListPosition(esBot.room.users[ind].id) + 1;
                esBot.userUtilities.updatePosition(esBot.room.users[ind], wlIndex);
            }
            esBot.room.afkInterval = setInterval(function(){esBot.roomUtilities.afkCheck()}, 10 * 1000);
            esBot.room.autodisableInterval = setInterval(function(){esBot.room.autodisableFunc();}, 60 * 60 * 1000);
            esBot.loggedInID = API.getUser().id;            
            esBot.status = true;
            API.sendChat('/cap 1');
            API.setVolume(0);
            API.sendChat('/me FarofinhaBot v1.1.21 ligado!');
        },                        
        commands: {        
            executable: function(minRank, chat){
                var id = chat.fromID;
                var perm = esBot.userUtilities.getPermission(id);
                var minPerm;
                switch(minRank){
                        case 'admin': minPerm = 7; break;
                        case 'ambassador': minPerm = 6; break;
                        case 'host': minPerm = 5; break;
                        case 'cohost': minPerm = 4; break;
                        case 'manager': minPerm = 3; break;
                        case 'mod': 
                                if(esBot.roomSettings.bouncerPlus){
                                    minPerm = 2;
                                }
                                else {
                                    minPerm = 3;
                                }
                                break;
                        case 'bouncer': minPerm = 2; break;
                        case 'residentdj': minPerm = 1; break;
                        case 'user': minPerm = 0; break;
                        default: API.chatLog('error assigning minimum permission');
                };
                if(perm >= minPerm){
                return true;
                }
                else return false;                      
            },                
                /**
                commandCommand: {
                        rank: 'user/bouncer/mod/manager',
                        type: 'startsWith/exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                
                                };                              
                        },
                },          
                **/

                activeCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var now = Date.now();
                                    var chatters = 0;
                                    var time;
                                    if(msg.length === cmd.length) time = 60;
                                    else{
                                        time = msg.substring(cmd.length + 1);
                                        if(isNaN(time)) return API.sendChat('/me [@' + chat.from + '] Invalid time specified.');
                                    }
                                    for(var i = 0; i < esBot.room.users.length; i++){
                                        userTime = esBot.userUtilities.getLastActivity(esBot.room.users[i]);
                                        if((now - userTime) <= (time * 60 * 1000)){
                                        chatters++;
                                        }
                                    }
                                    API.sendChat('/me [@' + chat.from + '] Tivemos ' + chatters + ' usuários conversando nos últimos ' + time + ' minutos.');
                                };                              
                        },
                },

                addCommand: {
                        rank: 'mod',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] No user specified.');
                                    var name = msg.substr(cmd.length + 2);   
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if (msg.length > cmd.length + 2) {
                                        if (typeof user !== 'undefined') {
                                            if(esBot.room.roomevent){
                                                esBot.room.eventArtists.push(user.id);
                                            }
                                            esBot.userUtilities.moveUser(user.id, 0, false);
                                        } else API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                      }
                                };                              
                        },
                },

                afklimitCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Limite não especificado');
                                    var limit = msg.substring(cmd.length + 1);
                                    if(!isNaN(limit)){
                                        esBot.roomSettings.maximumAfk = parseInt(limit, 10);
                                        API.sendChat('/me [@' + chat.from + '] Duração de afk agora é ' + esBot.roomSettings.maximumAfk + ' minutos.');
                                    }
                                    else API.sendChat('/me [@' + chat.from + '] Limite inválido.');
                                };                              
                        },
                },

                afkremovalCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.afkRemoval){
                                        esBot.roomSettings.afkRemoval = !esBot.roomSettings.afkRemoval;
                                        clearInterval(esBot.room.afkInterval);
                                        API.sendChat('/me [@' + chat.from + '] Não removendo afks.');
                                      }
                                    else {
                                        esBot.roomSettings.afkRemoval = !esBot.roomSettings.afkRemoval;
                                        esBot.room.afkInterval = setInterval(function(){esBot.roomUtilities.afkCheck()}, 2 * 1000);
                                        API.sendChat('/me [@' + chat.from + '] Removendo afks.');
                                      }
                                };                              
                        },
                },

                afkresetCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.')
                                    var name = msg.substring(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    esBot.userUtilities.setLastActivity(user);
                                    API.sendChat('/me [@' + chat.from + '] Status afk resetado: @' + name + '.');
                                };                              
                        },
                },

                afktimeCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{                                    
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.');
                                    var name = msg.substring(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    var lastActive = esBot.userUtilities.getLastActivity(user);
                                    var inactivity = Date.now() - lastActive;
                                    var time = esBot.roomUtilities.msToStr(inactivity);
                                    API.sendChat('/me [@' + chat.from + '] @' + name + ' esteve inativo por ' + time + '.');
                                };                              
                        },
                },
                
                autoskipCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.autoskip){
                                        esBot.roomSettings.autoskip = !esBot.roomSettings.autoskip;
                                        clearTimeout(esBot.room.autoskipTimer);
                                        return API.sendChat('/me [@' + chat.from + '] Autoskip desligado.');
                                    }
                                    else{
                                        esBot.roomSettings.autoskip = !esBot.roomSettings.autoskip;
                                        return API.sendChat('/me [@' + chat.from + '] Autoskip ligado.');
                                    }
                                };                              
                        },
                },

                autowootCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat("/me Recomendado PlugCubed para autowoot: http://plugcubed.net/")
                                };                              
                        },
                },

                baCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat("/me Um Embaixador de Marca é a voz dos usuários da plug.dj. Eles promovem eventos, ajudam as comunidades e divulgam a plug.dj para todo o mundo. Mais info: http://blog.plug.dj/brand-ambassadors/");
                                };                              
                        },
                },

                banCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.');
                                    var name = msg.substr(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    //API.sendChat('/me [' + chat.from + ' whips out the banhammer :hammer:]');
                                    API.moderateBanUser(user.id, 1, API.BAN.DAY);
                                };                              
                        },
                },

                bouncerPlusCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(esBot.roomSettings.bouncerPlus){
                                        esBot.roomSettings.bouncerPlus = false;
                                        return API.sendChat('/me [@' + chat.from + '] Bouncer+ desligado.');
                                        }
                                    else{ 
                                        if(!esBot.roomSettings.bouncerPlus){
                                            var id = chat.fromID;
                                            var perm = esBot.userUtilities.getPermission(id);
                                            if(perm > 2){
                                                esBot.roomSettings.bouncerPlus = true;
                                                return API.sendChat('/me [@' + chat.from + '] Bouncer+ ligado.');
                                            }
                                        }
                                        else return API.sendChat('/me [@' + chat.from + '] Você precisa ser manager ou mais para ligar Bouncer+.');
                                    };
                                };                              
                        },
                },

                clearchatCommand: {
                        rank: 'manager',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var currentchat = $('#chat-messages').children();       
                                    for (var i = 0; i < currentchat.length; i++) {
                                        for (var j = 0; j < currentchat[i].classList.length; j++) {
                                            if (currentchat[i].classList[j].indexOf('cid-') == 0) 
                                                API.moderateDeleteChat(currentchat[i].classList[j].substr(4));
                                        }
                                    }                                 
                                return API.sendChat('/me [@' + chat.from + '] :jack_o_lantern:');
                                
                                };                              
                        },
                },

                commandsCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat("/me "+ esBot.name + " commands: " + esBot.cmdLink);
                                };                              
                        },
                },

                cookieCommand: {
                        rank: 'user',
                        type: 'startsWith',

                        cookies: ['te deu um cookie de chocolate!',
                                   'te enviou uma caixa de cookies. Aproveite!!!',
                                   'te deu o último cookie do pacote... Que amigo, hein!',
                                   'te deu um cookie gorduroso. Se eu fosse você, não comeria. rsrs',
                                   'tentou te enviar uns cookies mas acabou comendo tudo no caminho... que guloso!',
                                   'te deu um cookie do terror. Ou você come e morre, ou você morre!!! :smiling_imp:',
                                   'te deu um cookie lindo. É melhor você apreciar antes de comer!',
                                   'te deu um cookie premiado. Ligue 0800 2014 500 2000 e receba já seu caminhão de produtos Avon!',
                                   'te deu um cookie da sorte. Tem escrito: "O futuro depende dos seus sonhos, então vá dormir"',
                                   'te deu um cookie da sorte. Tem escrito: "Uma dieta balanceada é um chocolate em cada mão"',
                                   'te deu um cookie da sorte. Tem escrito: "Siga os bons e aprenda com eles."',
                                   'te deu um cookie com pedaços de morango, yummyyyy!',
                                   'te deu um cookie enorme. Se você encostar no cookie, ele se transforma num copo de vodka.',
                                   'te deu um cookie da sorte. Tem escrito: "Porque não estás trabalhando?"',
                                   'te deu um cookie da sorte. Dentro há: "Cumprimente agora a pessoa amada"',
                                   'te deu um cookie da sorte. Tem escrito: "Ordináááááária!!!"',
                                   'te deu um cookie da sorte. Tem escrito: "Sai desse computador..."',
                                   'te deu um cookie da sorte. Tem escrito: "Não esqueça de comer verduras!"',
                                   'te deu um cookie da sorte. Tem escrito: "Saaaaamba!"',
                                   'te deu um cookie da sorte. Tem escrito: "Coma com carinho"',
                                   'te deu um cookie da sorte. Tem escrito: "Água mole em pedra dura, tanto bate até que fura."',
                                   'te deu um cookie da sorte. Tem escrito: "Eu te amo"',
                                   'te deu um cookie dourado. Você não pode comer porque é feito de ouro. Droga!',
                                   'te deu um cookie do azar. Agora volte ao fim da fila! :trollface:',
                                   'te deu um cookie do árco-íris, cheio de amor :heart:',
                                   'te deu um cookie que foi esquecido na chuva... tá mole! Eca.',
                                   'te deu um cookie. Coma rápido!!!',
                                   'te deu um coo... kie delicioso rsrs :3',
                                   'te deu um cookie bugado. Cuidado pois você p-pode ... asdkljsadgjdf *ERROR*'
                                   
                            ],

                        getCookie: function() {
                            var c = Math.floor(Math.random() * this.cookies.length);
                            return this.cookies[c];
                        },

                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
      
                                    var space = msg.indexOf(' ');
                                    if(space === -1){ 
                                        API.sendChat('/em :cookie: COOKIES, COOKIES!!! *om nom nom* :cookie:');
                                        return false;
                                    }
                                    else{
                                        var name = msg.substring(space + 2);
                                        var user = esBot.userUtilities.lookupUserName(name);
                                        if (user === false || !user.inRoom) {
                                          return API.sendChat("/em '" + name + "' não está na sala. Eu mesmo vou comer.");
                                        } 
                                        else if(user.username === chat.from){
                                            return API.sendChat("/me @" + name +  ", porque não divide comigo? Dar cookies a si mesmo é mesquinho! u_u")
                                        }
                                        else {
                                            return API.sendChat("/me @" + user.username + ", @" + chat.from + ' ' + this.getCookie() );
                                        }
                                    }
                                
                                };                              
                        },
                },

                cycleCommand: {
                        rank: 'manager',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    esBot.roomUtilities.changeDJCycle();
                                };                              
                        },
                },

                cycleguardCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.cycleGuard){
                                        esBot.roomSettings.cycleGuard = !esBot.roomSettings.cycleGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Cycleguard disabled.');
                                    }
                                    else{
                                        esBot.roomSettings.cycleGuard = !esBot.roomSettings.cycleGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Cycleguard enabled.');
                                    }
                                
                                };                              
                        },
                },

                cycletimerCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var cycleTime = msg.substring(cmd.length + 1);
                                    if(!isNaN(cycleTime)){
                                        esBot.roomSettings.maximumCycletime = cycleTime;
                                        return API.sendChat('/me [@' + chat.from + '] The cycleguard is set to ' + esBot.roomSettings.maximumCycletime + ' minute(s).');
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] No correct time specified for the cycleguard.');
                                
                                };                              
                        },
                },

                dclookupCommand: {
                        rank: 'user',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var name;
                                    if(msg.length === cmd.length) name = chat.from;
                                    else{ 
                                        name = msg.substring(cmd.length + 2);
                                        var perm = esBot.userUtilities.getPermission(chat.fromID);
                                        if(perm < 2) return API.sendChat('/me [@' + chat.from + '] Somente bouncers e acima podem usar !dclookup');
                                    }    
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    var id = user.id;
                                    var toChat = esBot.userUtilities.dclookup(id);
                                    API.sendChat(toChat);
                                };                              
                        },
                },

                emojiCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat('/me Emoji: http://www.emoji-cheat-sheet.com/');
                                };                              
                        },
                },

                englishCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(chat.message.length === cmd.length) return API.sendChat('/me Usuário não especificado.');
                                    var name = chat.message.substring(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me Usuário inválido.');
                                    var lang = esBot.userUtilities.getUser(user).language;
                                    var ch = '/me @' + name + ' ';
                                    switch(lang){
                                        case 'en': break;
                                        case 'da': ch += 'Vær venlig at tale engelsk.'; break;
                                        case 'de': ch += 'Bitte sprechen Sie Englisch.'; break;
                                        case 'es': ch += 'Por favor, hable Inglés.'; break;
                                        case 'fr': ch += 'Parlez anglais, s\'il vous plaît.'; break;
                                        case 'nl': ch += 'Spreek Engels, alstublieft.'; break;
                                        case 'pl': ch += 'Proszę mówić po angielsku.'; break;
                                        case 'pt': ch += 'Por favor, fale Inglês.'; break;
                                        case 'sk': ch += 'Hovorte po anglicky, prosím.'; break;
                                        case 'cs': ch += 'Mluvte prosím anglicky.'; break;
                                        case 'sr': ch += 'Молим Вас, говорите енглески.'; break;                                  
                                    }
                                    ch += ' Português, por favor.';
                                    API.sendChat(ch);
                                };                              
                        },
                },

                etaCommand: {
                        rank: 'user',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var perm = esBot.userUtilities.getPermission(chat.fromID);
                                    var msg = chat.message;
                                    var name;
                                    if(msg.length > cmd.length){
                                        if(perm < 2) return void (0);
                                        name = msg.substring(cmd.length + 2);
                                      } else name = chat.from;
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    var pos = API.getWaitListPosition(user.id);
                                    if(pos < 0) return API.sendChat('/me @' + name + ', você não está na fila de espera.');
                                    var timeRemaining = API.getTimeRemaining();
                                    var estimateMS = ((pos+1) * 4 * 60 + timeRemaining) * 1000;
                                    var estimateString = esBot.roomUtilities.msToStr(estimateMS);
                                    API.sendChat('/me @' + name + ' você tocará em aproximadamente ' + estimateString + '.');                       
                                };                              
                        },
                },

                fbCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.fbLink === "string")
                                        API.sendChat('/me [' + chat.from + '] Entre no nosso grupo no Facebook: ' + esBot.roomSettings.fbLink);
                                };                              
                        },
                },

                filterCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.filterChat){
                                        esBot.roomSettings.filterChat = !esBot.roomSettings.filterChat;
                                        return API.sendChat('/me [@' + chat.from + '] Filtro de chat desativado.');
                                    }
                                    else{
                                        esBot.roomSettings.filterChat = !esBot.roomSettings.filterChat;
                                        return API.sendChat('/me [@' + chat.from + '] Filtro de chat ativado.');
                                    } 
                                
                                };                              
                        },
                },

                joinCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.room.roulette.rouletteStatus){
                                        esBot.room.roulette.participants.push(chat.fromID);
                                        API.sendChat("/me @" + chat.from + " entrou na loteria! (digite !leave parar sair)");
                                    }
                                };                              
                        },
                },

                jointimeCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.');
                                    var name = msg.substring(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    var join = esBot.userUtilities.getJointime(user);
                                    var time = Date.now() - join;
                                    var timeString = esBot.roomUtilities.msToStr(time);
                                    API.sendChat('/me [@' + chat.from + '] @' + name + ' esteve na sala por ' + timeString + '.');
                                };                              
                        },
                },

                kickCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var lastSpace = msg.lastIndexOf(' ');
                                    var time;
                                    var name;
                                    if(lastSpace === msg.indexOf(' ')){
                                        time = 0.25;
                                        name = msg.substring(cmd.length + 2);
                                        }    
                                    else{
                                        time = msg.substring(lastSpace + 1);
                                        name = msg.substring(cmd.length + 2, lastSpace);
                                    }
                                    
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    var from = chat.from;
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário não válido.');

                                    var permFrom = esBot.userUtilities.getPermission(chat.fromID);
                                    var permTokick = esBot.userUtilities.getPermission(user.id);

                                    if(permFrom <= permTokick)
                                        return API.sendChat("/me [@" + chat.from + "] você não pode kickar usuários de mesmo nível ou maior que o seu!")

                                    if(!isNaN(time)){
                                        API.sendChat('/me [' + chat.from + ' usou kick, super efetivo!]');
                                        API.sendChat('[@' + name + '], você está sendo removido da sala por ' + time + ' minutos.');

                                        if(time > 24*60*60) API.moderateBanUser(user.id, 1 , API.BAN.PERMA);
                                            else API.moderateBanUser(user.id, 1, API.BAN.DAY);
                                        setTimeout(function(id, name){ 
                                            API.moderateUnbanUser(id); 
                                            console.log('Desbani @' + name + '.'); 
                                            }, time * 60 * 1000, user.id, name);
                                        
                                    }

                                    else API.sendChat('/me [@' + chat.from + '] Tempo inválido.');                                   
                                };                              
                        },
                },

                killCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    storeToStorage();
                                    API.sendChat('/me FarofinhaBot desligado.');
                                    esBot.disconnectAPI();
                                    setTimeout(function(){kill();},1000);
                                };                              
                        },
                },

                leaveCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var ind = esBot.room.roulette.participants.indexOf(chat.fromID);
                                    if(ind > -1){
                                        esBot.room.roulette.participants.splice(ind, 1);
                                        API.sendChat("/me @" + chat.from + " saiu da loteria!");
                                    }
                                };                              
                        },
                },

                linkCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var media = API.getMedia();
                                    var from = chat.from;
                                    var user = esBot.userUtilities.lookupUser(chat.fromID);
                                    var perm = esBot.userUtilities.getPermission(chat.fromID);
                                    var dj = API.getDJ().id;
                                    var isDj = false;
                                    if (dj === chat.fromID) isDj = true;
                                    if(perm >= 1 || isDj){
                                        if(media.format === '1'){
                                            var linkToSong = "https://www.youtube.com/watch?v=" + media.cid;
                                            API.sendChat('/me [' + from + '] Link da música atual: ' + linkToSong);
                                        }
                                        if(media.format === '2'){
                                            var SCsource = '/tracks/' + media.cid;
                                            SC.get('/tracks/' + media.cid, function(sound){API.sendChat('/me [' + from + '] Link da música atual: ' + sound.permalink_url);});
                                        }   
                                    }                    
                                
                                
                                };                              
                        },
                },

                lockCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    esBot.roomUtilities.booth.lockBooth();
                                };                              
                        },
                },

                lockdownCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var temp = esBot.roomSettings.lockdownEnabled;
                                    esBot.roomSettings.lockdownEnabled = !temp;
                                    if(esBot.roomSettings.lockdownEnabled){
                                        return API.sendChat("/me [@" + chat.from + "] Lockdown ativado. Somente staff pode conversar.");
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] Lockdown desativado.');
                                
                                };                              
                        },
                },

                lockguardCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.lockGuard){
                                        esBot.roomSettings.lockGuard = !esBot.roomSettings.lockGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Lockguard disabled.');
                                    }
                                    else{
                                        esBot.roomSettings.lockGuard = !esBot.roomSettings.lockGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Lockguard enabled.');
                                    } 
                                
                                };                              
                        },
                },

                lockskipCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.room.skippable){

                                        var dj = API.getDJ();
                                        var id = dj.id;
                                        var name = dj.username;
                                        var msgSend = '@' + name + ': ';

                                        esBot.room.queueable = false;

                                        if(chat.message.length === cmd.length){
                                            API.sendChat('/me [' + chat.from + ' usou lockskip.]');
                                            esBot.roomUtilities.booth.lockBooth();
                                            //esBot.roomUtilities.changeDJCycle();
                                            setTimeout(function(id){
                                                API.moderateForceSkip();
                                                esBot.room.skippable = false;
                                                setTimeout(function(){ esBot.room.skippable = true}, 5*1000);
                                                setTimeout(function(id){
                                                    esBot.userUtilities.moveUser(id, esBot.roomSettings.lockskipPosition, false);
                                                    esBot.room.queueable = true;
                                                    setTimeout(function(){esBot.roomUtilities.booth.unlockBooth();}, 1000);
                                                    //esBot.roomUtilities.changeDJCycle();
                                                    
                                                },1500, id);
                                            }, 1000, id);

                                            return void (0);

                                        }
                                        var validReason = false;
                                        var msg = chat.message;
                                        var reason = msg.substring(cmd.length + 1);       
                                        for(var i = 0; i < esBot.roomSettings.lockskipReasons.length; i++){
                                            var r = esBot.roomSettings.lockskipReasons[i][0];
                                            if(reason.indexOf(r) !== -1){
                                                validReason = true;
                                                msgSend += esBot.roomSettings.lockskipReasons[i][1];
                                            }
                                        }
                                        if(validReason){
                                            API.sendChat('/me [' + chat.from + ' lockskip.]');
                                            esBot.roomUtilities.booth.lockBooth();
                                            //esBot.roomUtilities.changeDJCycle();
                                            setTimeout(function(id){
                                                API.moderateForceSkip();
                                                esBot.room.skippable = false;
                                                API.sendChat(msgSend);
                                                setTimeout(function(){ esBot.room.skippable = true}, 5*1000);
                                                setTimeout(function(id){
                                                    esBot.userUtilities.moveUser(id, esBot.roomSettings.lockskipPosition, false);
                                                    esBot.room.queueable = true;
                                                    setTimeout(function(){esBot.roomUtilities.booth.unlockBooth();}, 1000);
                                                    //esBot.roomUtilities.changeDJCycle();
                                                    
                                                },1500, id);
                                            }, 1000, id);

                                            return void (0);
                                        }
                                                                                
                                    }
                                }                              
                        },
                },

                lockskipposCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var pos = msg.substring(cmd.length + 1);
                                    if(!isNaN(pos)){
                                        esBot.roomSettings.lockskipPosition = pos;
                                        return API.sendChat('/me [@' + chat.from + '] Lockskip está movendo agora para posição ' + esBot.roomSettings.lockskipPosition + '.');
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] Posição inválida.');
                                };                              
                        },
                },

                locktimerCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var lockTime = msg.substring(cmd.length + 1);
                                    if(!isNaN(lockTime)){
                                        esBot.roomSettings.maximumLocktime = lockTime;
                                        return API.sendChat('/me [@' + chat.from + '] The lockguard is set to ' + esBot.roomSettings.maximumLocktime + ' minute(s).');
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] No correct time specified for the lockguard.');
                                };                              
                        },
                },

                maxlengthCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var maxTime = msg.substring(cmd.length + 1);
                                    if(!isNaN(maxTime)){
                                        esBot.roomSettings.maximumSongLength = maxTime;
                                        return API.sendChat('/me [@' + chat.from + '] A duração de música agora é ' + esBot.roomSettings.maximumSongLength + ' minutes.');
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] Duração especificada inválida.');
                                };                              
                        },
                },

                motdCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length <= cmd.length + 1) return API.sendChat('/me MotD: ' + esBot.roomSettings.motd);
                                    var argument = msg.substring(cmd.length + 1);
                                    if(!esBot.roomSettings.motdEnabled) esBot.roomSettings.motdEnabled = !esBot.roomSettings.motdEnabled;
                                    if(isNaN(argument)){
                                        esBot.roomSettings.motd = argument;
                                        API.sendChat("/me MotD set to: " + esBot.roomSettings.motd);
                                    }
                                    else{
                                        esBot.roomSettings.motdInterval = argument;
                                        API.sendChat('/me Intervalo da MotD ' + esBot.roomSettings.motdInterval + '.');
                                    }
                                };                              
                        },
                },

                moveCommand: {
                        rank: 'mod',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.');
                                    var firstSpace = msg.indexOf(' ');
                                    //var secondSpace = msg.substring(firstSpace + 1).indexOf(' ');
                                    var lastSpace = msg.lastIndexOf(' ');
                                    var pos;
                                    var name;
                                    if(isNaN(parseInt(msg.substring(lastSpace + 1))) ){
                                        pos = 1;
                                        name = msg.substring(cmd.length + 2);
                                    }
                                    else{
                                        pos = parseInt(msg.substring(lastSpace + 1));
                                        name = msg.substring(cmd.length + 2,lastSpace);
                                    }
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    if(user.id === esBot.loggedInID) return API.sendChat('/me [@' + chat.from + '] Não me adicione a fila, por favor.');
                                    if (!isNaN(pos)) {
                                        API.sendChat('/me [' + chat.from + ' usou \'move\'.]');
                                        esBot.userUtilities.moveUser(user.id, pos, false); 
                                    } else return API.sendChat('/me [@' + chat.from + '] Posição inválida.');
                                };                           
                        },
                },

                muteCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Usuário não especificado.');
                                    var lastSpace = msg.lastIndexOf(' ');
                                    var time = null;
                                    var name;
                                    if(lastSpace === msg.indexOf(' ')){
                                        name = msg.substring(cmd.length + 2);
                                        }    
                                    else{
                                        time = msg.substring(lastSpace + 1);
                                        if(isNaN(time)){
                                            return API.sendChat('/me [@' + chat.from + '] Tempo inválido.');
                                        }
                                        name = msg.substring(cmd.length + 2, lastSpace);
                                    }
                                    var from = chat.from;
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(typeof user === 'boolean') return API.sendChat('/me [@' + chat.from + '] Usuário inválido.');
                                    var permFrom = esBot.userUtilities.getPermission(chat.fromID);
                                    var permUser = esBot.userUtilities.getPermission(user.id);
                                    if(permFrom > permUser){
                                        esBot.room.mutedUsers.push(user.id);
                                        if(time === null) API.sendChat('/me [@' + chat.from + '] Mutei @' + name + '.');
                                        else{
                                            API.sendChat('/me [@' + chat.from + '] Mutei @' + name + ' por ' + time + ' minutos.');
                                            setTimeout(function(id){
                                                var muted = esBot.room.mutedUsers;
                                                var wasMuted = false;
                                                var indexMuted = -1;
                                                for(var i = 0; i < muted.length; i++){
                                                    if(muted[i] === id){
                                                        indexMuted = i;
                                                        wasMuted = true;
                                                    }
                                                }
                                                if(indexMuted > -1){
                                                    esBot.room.mutedUsers.splice(indexMuted);
                                                    var u = esBot.userUtilities.lookupUser(id);
                                                    var name = u.username;
                                                    API.sendChat('/me [@' + chat.from + '] Desmutado @' + name + '.');
                                                }
                                            }, time * 60 * 1000, user.id);
                                        } 
                                    }
                                    else API.sendChat("/me [@" + chat.from + "] Você não pode mutar alguém do mesmo nível ou superior.");
                                };                              
                        },
                },

                opCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.opLink === "string")
                                        return API.sendChat("/me OP list: " + esBot.roomSettings.opLink);
                                    
                                };                              
                        },
                },

                pingCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat('/me Pong!')
                                };                              
                        },
                },

                refreshCommand: {
                        rank: 'manager',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    storeToStorage();
                                    esBot.disconnectAPI();
                                    setTimeout(function(){
                                    window.location.reload(false);
                                        },1000);
                                
                                };                              
                        },
                },

                reloadCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat('/me Reiniciando...');
                                    esBot.disconnectAPI();
                                    kill();
                                    setTimeout(function(){$.getScript(esBot.scriptLink);},2000);
                                };                              
                        },
                },

                removeCommand: {
                        rank: 'mod',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if (msg.length > cmd.length + 2) {
                                        var name = msg.substr(cmd.length + 2);
                                        var user = esBot.userUtilities.lookupUserName(name);
                                        if (typeof user !== 'boolean') {
                                            user.lastDC = {
                                                time: null,
                                                position: null,
                                                songCount: 0,
                                            };
                                            API.moderateRemoveDJ(user.id);                                          
                                        } else API.sendChat("/me [@" + chat.from + "] Usuário @" + name + " não está na fila de espera.");
                                      } else API.sendChat("/me [@" + chat.from + "] Usuário não especificado.");
                                
                                };                              
                        },
                },

                restrictetaCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.etaRestriction){
                                        esBot.roomSettings.etaRestriction = !esBot.roomSettings.etaRestriction;
                                        return API.sendChat('/me [@' + chat.from + '] ETA não restrito.');
                                    }
                                    else{
                                        esBot.roomSettings.etaRestriction = !esBot.roomSettings.etaRestriction;
                                        return API.sendChat('/me [@' + chat.from + '] ETA restrito.');
                                    } 
                                
                                };                              
                        },
                },

                rouletteCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(!esBot.room.roulette.rouletteStatus){
                                        esBot.room.roulette.startRoulette();
                                    }
                                };                              
                        },
                },

                rulesCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.rulesLink === "string")
                                        return API.sendChat("/me Regras aqui: " + esBot.roomSettings.rulesLink);                                
                                };                              
                        },
                },

                sessionstatsCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var from = chat.from;
                                    var woots = esBot.room.roomstats.totalWoots;
                                    var mehs = esBot.room.roomstats.totalMehs;
                                    var grabs = esBot.room.roomstats.totalCurates;
                                    API.sendChat('/me [@' + from + '] Woots: ' + woots + ', Mehs: ' + mehs + ', Grabs: ' + grabs + '.');
                                };                              
                        },
                },

                skipCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat('/me [' + chat.from + ' pulou a música.]');
                                    API.moderateForceSkip();
                                    esBot.room.skippable = false;
                                    setTimeout(function(){ esBot.room.skippable = true}, 5*1000);
                                
                                };                              
                        },
                },  

                sourceCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    API.sendChat('/me TBRBot criado por ninguém ' + esBot.creator + ' - ');
                                };                              
                        },
                },

                statusCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var from = chat.from
                                    var msg = '/me [@' + from + '] ';
                                      
                                    msg += 'Remoção de AFKs: ';
                                    if(esBot.roomSettings.afkRemoval) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';
                                    msg += "AFKs removidos: " + esBot.room.afkList.length + '. ';
                                    msg += 'Limite AFK: ' + esBot.roomSettings.maximumAfk + '. ';
                                     
                                    msg+= 'Bouncer+: '
                                    if(esBot.roomSettings.bouncerPlus) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';

                                    msg+= 'Lockguard: '
                                    if(esBot.roomSettings.lockGuard) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';

                                    msg+= 'Cycleguard: '
                                    if(esBot.roomSettings.cycleGuard) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';

                                    msg+= 'Timeguard: '
                                    if(esBot.roomSettings.timeGuard) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';

                                    msg+= 'Filtro: '
                                    if(esBot.roomSettings.filterChat) msg += 'ON';
                                    else msg += 'OFF';
                                    msg += '. ';

                                    var launchT = esBot.room.roomstats.launchTime;
                                    var durationOnline = Date.now() - launchT;
                                    var since = esBot.roomUtilities.msToStr(durationOnline);
                                    msg += 'Estou sambando por ' + since + '. ';
                                      
                                     return API.sendChat(msg);
                                };                              
                        },
                },
                
                sambarCommand: {
                        rank: 'mod',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('/me [@' + chat.from + '] Não deu pra sambar.');
                                    var firstSpace = msg.indexOf(' ');
                                    //var secondSpace = msg.substring(firstSpace + 1).indexOf(' ');
                                    var lastSpace = msg.lastIndexOf(' ');
                                    var name1 = msg.substring(cmd.length + 2,lastSpace);
                                    var name2 = msg.substring(lastSpace + 2);
                                    var user1 = esBot.userUtilities.lookupUserName(name1);
                                    var user2 = esBot.userUtilities.lookupUserName(name2);
                                    if(typeof user1 === 'boolean' || typeof user2 === 'boolean') return API.sendChat('/me [@' + chat.from + '] Samba inválido (espaço não permitido)');
                                    if(user1.id === esBot.loggedInID || user2.id === esBot.loggedInID) return API.sendChat('/me [@' + chat.from + '] Não sei sambar, só sei cantar...');
                                    var p1 = API.getWaitListPosition(user1.id);
                                    var p2 = API.getWaitListPosition(user2.id);
                                    if(p1 < 0 || p2 < 0) return API.sendChat('/me [@' + chat.from + '] Use \'sambar\'somente com usuários na fila de espera.');
                                    API.sendChat("/me Sambando posições: " + name1 + " com " + name2 + ".");
                                    if(p1 < p2){
                                        esBot.userUtilities.moveUser(user2.id, p1+1, false);
                                        setTimeout(function(user1, p2){
                                            esBot.userUtilities.moveUser(user1.id, p2+1, false);
                                        },2000, user1, p2);
                                    }
                                    else{
                                        esBot.userUtilities.moveUser(user1.id, p2+1, false);
                                        setTimeout(function(user2, p1){
                                            esBot.userUtilities.moveUser(user2.id, p1+1, false);
                                        },2000, user2, p1);
                                    }
                                };                           
                        },
                },

                themeCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.themeLink === "string")
                                        API.sendChat("/me Please find the permissible room genres here: " + esBot.roomSettings.themeLink);
                                
                                };                              
                        },
                },

                timeguardCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.timeGuard){
                                        esBot.roomSettings.timeGuard = !esBot.roomSettings.timeGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Timeguard disabled.');
                                    }
                                    else{
                                        esBot.roomSettings.timeGuard = !esBot.roomSettings.timeGuard;
                                        return API.sendChat('/me [@' + chat.from + '] Timeguard enabled.');
                                    } 
                                
                                };                              
                        },
                },

                togglemotdCommand: {
                        rank: 'bouncer',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.motdEnabled){
                                        esBot.roomSettings.motdEnabled = !esBot.roomSettings.motdEnabled;
                                        API.sendChat('/me MotD desativada.');
                                    }
                                    else{
                                        esBot.roomSettings.motdEnabled = !esBot.roomSettings.motdEnabled;
                                        API.sendChat('/me MotD ativada.');
                                    }
                                };                              
                        },
                },

                unbanCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    $(".icon-population").click();
                                    $(".icon-ban").click();
                                    setTimeout(function(chat){
                                        var msg = chat.message;
                                        if(msg.length === cmd.length) return API.sendChat()
                                        var name = msg.substring(cmd.length + 2);
                                        var bannedUsers = API.getBannedUsers();
                                        var found = false;
                                        for(var i = 0; i < bannedUsers.length; i++){
                                            var user = bannedUsers[i];
                                            if(user.username === name){
                                                id = user.id;
                                                found = true;
                                            }
                                          }
                                        if(!found){
                                            $(".icon-chat").click();
                                            return API.sendChat('/me [@' + chat.from + '] O usuário não foi banido.');  
                                        }                                
                                        API.moderateUnbanUser(user.id);
                                        console.log("Unbanned " + name);
                                        setTimeout(function(){
                                            $(".icon-chat").click();
                                        },1000);
                                    },1000,chat);
                                };                              
                        },
                },

                unlockCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    esBot.roomUtilities.booth.unlockBooth();
                                };                              
                        },
                },

                unmuteCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var permFrom = esBot.userUtilities.getPermission(chat.fromID);
                                      
                                    if(msg.indexOf('@') === -1 && msg.indexOf('all') !== -1){
                                        if(permFrom > 2){
                                            esBot.room.mutedUsers = [];
                                            return API.sendChat('/me [@' + chat.from + '] Todos foram desmutados.');
                                        }
                                        else return API.sendChat('/me [@' + chat.from + '] Somente coordenadores podem desmutar de uma vez.')
                                    }
                                      
                                    var from = chat.from;
                                    var name = msg.substr(cmd.length + 2);

                                    var user = esBot.userUtilities.lookupUserName(name);
                                      
                                    if(typeof user === 'boolean') return API.sendChat("/me Usuário inválido.");
                                    
                                    var permUser = esBot.userUtilities.getPermission(user.id);
                                    if(permFrom > permUser){

                                        var muted = esBot.room.mutedUsers;
                                        var wasMuted = false;
                                        var indexMuted = -1;
                                        for(var i = 0; i < muted.length; i++){
                                            if(muted[i] === user.id){
                                                indexMuted = i;
                                                wasMuted = true;
                                            }

                                        }
                                        if(!wasMuted) return API.sendChat("/me [@" + chat.from + "] o usuário não foi mutado.");
                                        esBot.room.mutedUsers.splice(indexMuted);
                                        API.sendChat('/me [@' + chat.from + '] Desmutado @' + name + '.');
                                    }
                                    else API.sendChat("/me [@" + chat.from + "] Você não pode mutar alguém do mesmo nível ou superior.");
                                    
                                };                              
                        },
                },

                usercmdcdCommand: {
                        rank: 'manager',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    var cd = msg.substring(cmd.length + 1);
                                    if(!isNaN(cd)){
                                        esBot.roomSettings.commandCooldown = cd;
                                        return API.sendChat('/me [@' + chat.from + '] The cooldown for commands by users is now set to ' + esBot.roomSettings.commandCooldown + ' seconds.');
                                    }
                                    else return API.sendChat('/me [@' + chat.from + '] No correct cooldown specified.');
                                
                                };                              
                        },
                },

                usercommandsCommand: {
                        rank: 'manager',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.usercommandsEnabled){
                                        API.sendChat('/me [@' + chat.from + '] Usercommands disabled.');
                                        esBot.roomSettings.usercommandsEnabled = !esBot.roomSettings.usercommandsEnabled;
                                    }
                                    else{
                                        API.sendChat('/me [@' + chat.from + '] Usercommands enabled.');
                                        esBot.roomSettings.usercommandsEnabled = !esBot.roomSettings.usercommandsEnabled;
                                    }
                                };                              
                        },
                },

                voteratioCommand: {
                        rank: 'bouncer',
                        type: 'startsWith',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    var msg = chat.message;
                                    if(msg.length === cmd.length) return API.sendChat('[@' + chat.from + '] No user specified.');
                                    var name = msg.substring(cmd.length + 2);
                                    var user = esBot.userUtilities.lookupUserName(name);
                                    if(user === false) return API.sendChat('/me [@' + chat.from + '] Invalid user specified.');
                                    var vratio = user.votes;
                                    var ratio = vratio.woot / vratio.meh;
                                    API.sendChat('/me [@' + chat.from + '] @' + name + ' ~ woots: ' + vratio.woot + ', mehs: ' + vratio.meh + ', ratio (w/m): ' + ratio.toFixed(2) + '.');
                                };                              
                        },
                },

                welcomeCommand: {
                        rank: 'mod',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(esBot.roomSettings.welcome){
                                        esBot.roomSettings.welcome = !esBot.roomSettings.welcome;
                                        return API.sendChat('/me [@' + chat.from + '] Mensagem de boas vindas desativada.');
                                    }
                                    else{
                                        esBot.roomSettings.welcome = !esBot.roomSettings.welcome;
                                        return API.sendChat('/me [@' + chat.from + '] Mensagem de boas vindas ativada.');
                                    } 
                                
                                };                              
                        },
                },

                websiteCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.website === "string")
                                        API.sendChat('/me Visite nossa página: ' + esBot.roomSettings.website);
                                };                              
                        },
                },

                youtubeCommand: {
                        rank: 'user',
                        type: 'exact',
                        functionality: function(chat, cmd){
                                if(this.type === 'exact' && chat.message.length !== cmd.length) return void (0);
                                if( !esBot.commands.executable(this.rank, chat) ) return void (0);
                                else{
                                    if(typeof esBot.roomSettings.youtubeLink === "string")
                                        API.sendChat('/me [' + chat.from + '] Subscribe to us on youtube: ' + esBot.roomSettings.youtubeLink);                                
                                };                              
                        },
                },
                
        },
                
};

esBot.startup(); 
}).call(this);
