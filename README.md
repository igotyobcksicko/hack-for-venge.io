download tamper monkey 
https://www.tampermonkey.net/
script 1
// ==UserScript==
// @name         Venge.io AIMBOT/ESP/TELEPORT hacks WORKING 1.0.26
// @version      0.71
// @description  Venge.io AIMBOT/ESP/TELEPORT
// @author       MetaRobot
// @match        https://venge.io/*
// @grant       GM_xmlhttpRequest
// @grant       unsafeWindow
// @run-at      document-start
// @namespace https://greasyfork.org/users/663589
// ==/UserScript==


// oh noes, bye bye game
unsafeWindow.document.documentElement.innerHTML = ``;

function processSource(gameSource){

   // Remove pathetic attempt at anticheat, lol
   let newSource = gameSource;
   newSource = newSource.replace(/VengeGuard\.prototype\.onCheck=function\(\){[\s\S]+?(?=})/, `VengeGuard.prototype.onCheck = function() { /* more like vengetard LOL */ this.app.fire("Network:Guard",true);`);
   newSource = newSource.replace(`["guard",e]`, `["guard",true]`); //awww this one was kinda cute hope nobody's getting paid for this

   // ESP (im even leaving comments!! good luck guys)
   newSource = newSource.replace(`Date.now()-this.player.lastDamage>1800`, `Date.now()-this.player.lastDamage>1800&&!window.VengeTard.esp`);

   return newSource;
}


function processEngine(engineSource){ // Not needed, yet...
     let newSource = engineSource;
     return newSource;
}


let getD3D = function(a, b, c, d, e, f) { // but muh pLaYeRcaNvAs.UtIlS
        let g = a - d, h = b - e, i = c - f;
        return Math.sqrt(g * g + h * h + i * i);
    };
let getXDire = function(a, b, c, d, e, f) {
    let g = Math.abs(b - e), h = getD3D(a, b, c, d, e, f);
    return Math.asin(g / h) * (b > e ? -1 : 1);
};

// wow this kinda sucks for you doesnt it
// maybe add a check for.. hmm yikes
// maybe dont use shitty game maker software???
GM_xmlhttpRequest({
    method: "GET",
    url: `https://venge.io/playcanvas-stable.min.js`, // Fucking idiots
    onload: jsresp => {
        let code = jsresp.responseText;

        GM_xmlhttpRequest({
            method: "GET" ,
            url: document.location.origin,
            onload: inRes => {
                let dbody = inRes.responseText;
                let newBody = dbody;

                let newCode = processEngine(code);

                newBody = newBody.replace(/<script src="playcanvas-stable\.min\.js"><\/script>/, `<script>${newCode}</script>`)

                document.open();
                unsafeWindow.document.write(newBody);
                document.close();

                document.oldCreateElement = document.createElement;
                unsafeWindow.loadCallback = false;
                document.createElement = function(args){
                    let retVal = this.oldCreateElement(...arguments);
                    retVal.__defineSetter__("src", function(src){
                        if (src.includes("__game-scripts.js")){ // uh oh might wanna patch this!
                            unsafeWindow.loadCallback = retVal.onload;
                            fetch(`${document.baseURI}${src}`).then(res=>res.text()).then(gameSource => {
                                let procSource = processSource(gameSource);
                                unsafeWindow.eval(procSource); // hmmmm, and this
                                unsafeWindow.loadCallback();
                            });
                        } else {
                            retVal.setAttribute("src", src); // :(
                        }
                    });
                    return retVal;
                }

                // Ohh nowowoo im so scawed is venge modeator #0492 going to ban me from teh discowddd!! owowoo
                unsafeWindow.VengeTard = {kill: true, esp: true};
                unsafeWindow.VengeTard.tick = function() {
                    if (unsafeWindow.VengeTard.kill) {
                        var closest;
                        var rec;

                        var players = unsafeWindow.VengeTard.network.players;
                        for (var i = 0; i < players.length; i++) {
                            var target = players[i];
                            var t = unsafeWindow.VengeTard.movement.entity.getPosition();
                            let calcDist = Math.sqrt( (target.position.y-t.y)**2 + (target.position.x-t.x)**2 + (target.position.z-t.z)**2 );
                            if (calcDist < rec || !rec) {
                                closest = target;
                                rec = calcDist;
                            }
                        }

                        // if (window.closestp) hack(); "iMpRoved ANTiChEat"
                        unsafeWindow.closestp = closest;
                        let rayCastList = pc.app.systems.rigidbody.raycastAll(unsafeWindow.VengeTard.movement.entity.getPosition(), closestp.getPosition()).map(x=>x.entity.tags._list.toString())
                        let rayCastCheck = rayCastList.length === 1 && rayCastList[0] === "Player";
                        if (closest && rayCastCheck) {
                            t = unsafeWindow.VengeTard.movement.entity.getPosition()
                                , e = Utils.lookAt(closest.position.x, closest.position.z, t.x, t.z);

                            // Oh fuck oh shit they found Math.random we're fucked
                            unsafeWindow.VengeTard.movement.lookX = e * 57.29577951308232 + Math.random()/10 - Math.random()/10;
                            unsafeWindow.VengeTard.movement.lookY = -1 * (getXDire(closest.position.x, closest.position.y, closest.position.z, t.x, t.y+0.9, t.z)) * 57.29577951308232;
                            unsafeWindow.VengeTard.movement.leftMouse = true;
                            unsafeWindow.VengeTard.movement.setShooting(unsafeWindow.VengeTard.movement.lastDelta);
                        } else {
                           unsafeWindow.VengeTard.movement.leftMouse = false;
                        }
                    }
                };

                (async() => {
                    while(!unsafeWindow.hasOwnProperty("Movement"))
                        await new Promise(resolve => setTimeout(resolve, 1000));

                    var updateHooked = false;
                    const update = Movement.prototype.update;
                    Movement.prototype.update = function (t) {
                        if (!updateHooked) {
                            unsafeWindow.VengeTard.movement = this;
                            updateHooked = true;
                        }
                        unsafeWindow.VengeTard.tick();
                        update.apply(this, [t]);

                    };
                })();

                (async() => {
                    while(!unsafeWindow.hasOwnProperty("NetworkManager"))
                        await new Promise(resolve => setTimeout(resolve, 1000));
                    var initializeHooked = false;
                    var manager = NetworkManager.prototype.initialize;
                    NetworkManager.prototype.initialize = function() {
                        if (!initializeHooked) {
                            unsafeWindow.VengeTard.network = this;
                            console.log(this)
                            initializeHooked = true;
                        }
                        manager.call(this);
                        unsafeWindow.VengeTard.network.app.fire("Chat:Message", "Hacks", "Welcome to VengeBOT Pro! SLASH = toggle aimbot, r = teleport to safety, t = ESP", !0);
                    };
                   })();

                // Add a check for window.VengeTard u wont
                unsafeWindow.addEventListener("keydown", function(e) {
						 if (e.keyCode == 191) { // SLASH
								 unsafeWindow.VengeTard.kill = !unsafeWindow.VengeTard.kill;
								 unsafeWindow.VengeTard.network.app.fire("Chat:Message", "Hacks", "Aimbot - " + (unsafeWindow.VengeTard.kill?"Enabled":"Disabled"), !0)
						 }

                         else if (e.keyCode == 82){ // R
                                  unsafeWindow.VengeTard.network.app.fire("Chat:Message", "Hacks", "You pressed R - Teleporting you to Safety", !0);
                                  unsafeWindow.VengeTard.movement.app.fire("Player:Respawn", !0);
                          }

                        else if (e.keyCode == 84){ // T
                             unsafeWindow.VengeTard.esp = !unsafeWindow.VengeTard.esp;
                             unsafeWindow.VengeTard.network.app.fire("Chat:Message", "Hacks", "ESP - " + (unsafeWindow.VengeTard.esp?"Enabled":"Disabled"), !0)

                        }
				 })

               }
        });
}});
