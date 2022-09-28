<template>
    <div :id="'glassCanvas-' + canvas_no" class="glass-break-canvas"></div>
</template>

<script>
    export default {
        name: 'shardCanvas',
        props: {
            canvas_no: Number,
            canvas_id: String,
            shard_cnt: Number,
            shard_size: Number,
            plays_audio: Boolean

        },
        data() {
            return {
                canvasedSketch : "",
            }
        },
        mounted() {
            /* ######### P5 stuff ######### */

            let style_rules = getComputedStyle(document.body);
            let props = this.$props;
            let glass_color = style_rules.getPropertyValue("--clear-glass-light");
            let chaffSize = 3;
            let chaffMultiplier = 3.5;
            let variance = 20;
            // Max preset horizontal velocity.
            let MPH_velocity = 4;
            // Max preset horizontal velocity.
            let MPV_velocity = 10;
            let g_acceleration = 0.6;
            let maxVelocity = 9.8;
            let crack_offset = {
                x: -8,
                y: -9
            };
            let canvas_dims = {
                x: 400,
                y: 1000
            };
            let canvas_offset = 100;
            let canvasShards = [];

            // Example for creating multiple p5 canvases.
            // https://editor.p5js.org/caminofarol/sketches/r609C2cs

            p5.disableFriendlyErrors = true;

            let newSketch = function (sketch) {
                let canvas_data_id = props.canvas_id;
                let canvas_data_parent = 'glassCanvas-' + props.canvas_no;
                let canvas_data_shard_cnt = props.shard_cnt;
                let canvas_data_shard_size = props.shard_size;
                let fill_c = sketch.color(glass_color);
                let plays_audio = props.plays_audio;
                let canvasElement;
                let distFromTop = 0;
                let lowerBound = canvas_dims.y;

                // We use both these variables to make sure the audio plays
                let groundAudioTriggers = 0;
                let groundAudioHasPlayed = false;
                
                sketch.clicks = 0;
                sketch.triggerMakeShards = false;

                sketch.setup = function () {
                    let glassCanvas = sketch.createCanvas(canvas_dims.x, canvas_dims.y);
                    glassCanvas.id(canvas_data_id);
                    glassCanvas.parent(canvas_data_parent);
                    canvasElement = document.querySelector("#" + canvas_data_parent + " .canvas");
                };

                sketch.draw = function () {
                    if (sketch.triggerMakeShards) {
                        sketch.makeShards()
                        sketch.triggerMakeShards = false;
                    }
                    // return;

                    if(groundAudioTriggers > 0 && groundAudioHasPlayed != true && plays_audio) {
                        // Make sure the user has clicked the dom and is intending to interact with the element that makes this sound.
                        if (sketch.clicks > 0) {
                            let groundHit = new Audio(
                                "src/assets/audio/GlassGroundHit.mp3"
                            );
                            groundHit.volume = 0.2;
                            groundHit.play();
                        }

                        groundAudioHasPlayed = true;
                    }

                    sketch.clear();
                    sketch.fill(fill_c);
                    sketch.strokeWeight(1);
                    sketch.stroke(sketch.color("rgba(255,255,255,0.8)"));
                    if (canvasShards.length > 0) {

                        // We invert the array to stop jittering.
                        canvasShards.reverse().forEach(function (element, SHindex) {
                            if (element.deletable) {
                                canvasShards.splice(SHindex, 1);

                                // On the first deletion, trigger the ground hit audio to play.
                                groundAudioTriggers++;
                            }
                        });

                        canvasShards.forEach(function (element, SHindex) {
                            element.update(lowerBound);
                            element.display(sketch);
                        });
                    }
                    else {

                        // When we dont need to draw, we should end the loop.
                        // https://p5js.org/reference/#/p5/loop
                        sketch.noLoop();
                    }
                };

                sketch.makeShards = function () {
                    // Reset the ground audio play var so we can hear the ground audio again.
                    groundAudioTriggers = 0;
                    groundAudioHasPlayed = false;

                    // Preliminary lower bound for the canvas.
                    canvasElement = document.querySelector("#" + canvas_data_parent + " canvas");
                    distFromTop = window.pageYOffset + canvasElement.getBoundingClientRect().top;
                    lowerBound = window.innerHeight - distFromTop + props.shard_size;

                    // if the lower bound is greater than the canvas has height...
                    if (lowerBound > canvas_dims.y) {
                        lowerBound = canvas_dims.y + props.shard_size;
                    }

                    canvasShards = shardsCreate(
                        canvas_data_shard_cnt,
                        canvas_data_shard_size,
                        lowerBound,
                        myCustomVertices
                    );
                    console.log(canvasShards);
                };
            };

            // Set up the canvases.
            this.canvasedSketch = new p5(newSketch);

            function shardsCreate(shard_cnt, shard_size, lowerBound, customShards = null) {
                let shards = [];
                if (customShards !== null) {
                    for (let i = 0; i < customShards.length; i++) {
                        let adjShard = [];
                        let posX = customShards[i][0][0];
                        let posY = customShards[i][0][1];
                        customShards[i].forEach(function(vertices, verticesIndex) {
                            adjShard[verticesIndex] = [];
                            adjShard[verticesIndex][0] = customShards[i][verticesIndex][0] - customShards[i][0][0];
                            adjShard[verticesIndex][1] = customShards[i][verticesIndex][1] - customShards[i][0][1];
                        });
                        
                        console.log(posX);
                        shards.push(
                            new Shard(
                                posX,
                                posY,
                                null,
                                lowerBound,
                                adjShard
                            )
                        );
                    }
                }
                else {
                    for (let i = 0; i < shard_cnt; i++) {
                        let ranposX = Math.floor(Math.random() * 50) - 25;
                        let ranposY = Math.floor(Math.random() * 50) - 50;
                        shards.push(
                            new Shard(
                                canvas_dims.x / 2 + ranposX - 10,
                                canvas_offset + ranposY,
                                shard_size,
                                lowerBound
                            )
                        );
                    }
                    for (let i = 0; i < shard_cnt * chaffMultiplier; i++) {
                        let ranposX = Math.floor(Math.random() * 50) - 25;
                        let ranposY = Math.floor(Math.random() * 50) - 50;
                        shards.push(
                            new Shard(
                                canvas_dims.x / 2 + ranposX - 10,
                                canvas_offset + ranposY,
                                null,
                                lowerBound
                            )
                        );
                    }
                }
                console.log(shards[0]);

                return shards;
            }

            // https://www.w3schools.com/js/js_object_constructors.asp
            // https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_Objects
            function Shard(iPosX, iPosY, size, lowerBound, customVertices = null) {
                // Variables.
                this.pos = {
                    x: iPosX,
                    y: iPosY
                };
                this.countDown = 100;

                this.chaff = true;
                if (size !== null && size > chaffSize) {
                    this.chaff = false;
                }

                this.lowerBound = lowerBound

                // Maximum rotation speed.
                // this.MR_speed = 3;
                // Rotation per frame.
                // this.rpf =
                //     Math.floor(Math.random() * (this.MR_speed * 2 + 1)) - this.MR_speed;
                // Current rotation.
                // this.rotation = 0;
                this.deletable = false;

                // this.vertices;

                if (customVertices !== null) {
                    this.vertices = [];
                    this.vertices[0] = customVertices;
                    this.vertices[1] = [0, 0];
                }
                else {
                    this.shard_variance = Math.floor(Math.random() * variance) - variance / 2;

                    // https://www.w3schools.com/js/js_random.asp
                    this.vertices_cnt = Math.floor(Math.random() * 3) + 3;
                    if (this.chaff === true) {
                        this.vertices = sortPoints(
                            randomPoints(this.vertices_cnt, chaffSize, chaffSize),
                            chaffSize,
                            chaffSize
                        );
                    } else {
                        this.vertices = sortPoints(
                            randomPoints(
                                this.vertices_cnt,
                                props.shard_size + this.shard_variance,
                                props.shard_size + this.shard_variance
                            ),
                            props.shard_size,
                            props.shard_size
                        );
                    }
                }
                
                this.horizontalVelocity = Math.random() * MPH_velocity - MPH_velocity / 2;
                this.verticalVelocity = Math.random() * MPV_velocity - MPV_velocity / 2;

                // Functions.
                this.update = function () {
                    if (this.countDown > 0) {
                        this.countDown--;
                    }
                    else {
                        this.verticalVelocity += g_acceleration;
                        if (this.verticalVelocity >= maxVelocity) {
                        this.verticalVelocity = maxVelocity;
                        }
                        // this.rotation += this.rpf;

                        this.pos = {
                            x: this.pos.x + this.horizontalVelocity,
                            y: this.pos.y + this.verticalVelocity
                        };

                        if (this.pos.y > this.lowerBound) {
                            this.deletable = true;
                        }
                    }
                };

                this.display = function (sketch) {
                    // if (this.chaff) {
                    //   return;
                    // }

                    sketch.push();

                    // DEBUG::SHOWS OFFSET CENTER.
                    // sketch.circle(
                    //   canvas_dims.x / 2 + crack_offset.x,
                    //   canvas_offset + crack_offset.y,
                    //   10
                    // );

                    // sketch.angleMode(sketch.DEGREES);
                    sketch.translate(
                        this.vertices[1][0] + this.pos.x + crack_offset.x,
                        this.vertices[1][1] + this.pos.y + crack_offset.y
                    );

                    // DEBUG::SHOWS ROTATION CENTER.
                    // circle(0, 0, 10);
                    // sketch.rotate(this.rotation);
                    sketch.translate(-this.vertices[1][0], -this.vertices[1][1]);
                    // sketch.circle(0,0,100);
                    sketch.beginShape();
                    for (let i = 0; i < this.vertices[0].length; i++) {
                        sketch.vertex(this.vertices[0][i][0], this.vertices[0][i][1]);
                    }
                    sketch.endShape(sketch.CLOSE);
                    sketch.pop();
                };
            }

            // BIG props.
            // https://observablehq.com/@tarte0/generate-random-simple-polygon
            function randomPoints(n, width, height) {
            const points = [];
            for (let i = 0; i < n; i++) {
                points.push([Math.random() * width, Math.random() * height]);
            }
            return points;
            }

            function sortPoints(points, width, height) {
            const centerPoint = [width / 2, height / 2];
            const sorted = points.slice(0);
            let angleFromCenter;

            const sortByAngle = (p1, p2) => {
                return (
                (Math.atan2(p1[1] - centerPoint[1], p1[0] - centerPoint[0]) * 180) /
                    Math.PI -
                (Math.atan2(p2[1] - centerPoint[1], p2[0] - centerPoint[0]) * 180) /
                    Math.PI
                );
            };

            sorted.sort(sortByAngle);
            return [sorted, centerPoint];
            }

        },
        methods: {
            makeShards(clicks) {
                this.canvasedSketch.triggerMakeShards = true;
                this.canvasedSketch.clicks = clicks;
                this.canvasedSketch.loop();
            }
        }
    }

    let myCustomVertices = [
        [ 
            [363.2, 338.14], 
            [369.6, 333.9], 
            [373.6, 328.6], 
        ], 
        [ 
            [369.6, 333.9], 
            [371.2, 334.96], 
            [373.6, 328.6], 
        ], 
        [ 
            [371.2, 334.96], 
            [372.8, 340.26], 
            [373.6, 328.6], 
        ], 
        [ 
            [372.8, 340.26], 
            [376, 341.32], 
            [373.6, 328.6], 
        ], 
        [ 
            [376, 341.32], 
            [374.4, 352.45000000000005], 
            [373.6, 328.6], 
        ], 
        [ 
            [374.4, 352.45000000000005], 
            [378.8, 350.86], 
            [373.6, 328.6], 
        ], 
        [ 
            [378.8, 350.86], 
            [379.84, 345.56], 
            [373.6, 328.6], 
        ], 
        [ 
            [379.84, 345.56], 
            [385.6, 345.56], 
            [373.6, 328.6], 
        ], 
        [ 
            [385.6, 345.56], 
            [387.2, 340.26], 
            [373.6, 328.6], 
        ], 
        [ 
            [387.2, 340.26], 
            [399.2, 332.84], 
            [373.6, 328.6], 
        ], 
        [ 
            [399.2, 332.84], 
            [400, 327.54], 
            [373.6, 328.6], 
        ], 
        [ 
            [400, 327.54], 
            [401.6, 324.36], 
            [373.6, 328.6], 
        ], 
        [ 
            [401.6, 324.36], 
            [401.99999999999994, 322.24], 
            [373.6, 328.6], 
        ], 
        [ 
            [401.99999999999994, 322.24], 
            [395.2, 319.59], 
            [373.6, 328.6], 
        ], 
        [ 
            [395.2, 319.59], 
            [390.8, 316.40999999999997], 
            [373.6, 328.6], 
        ], 
        [ 
            [390.8, 316.40999999999997], 
            [388, 323.3], 
            [373.6, 328.6], 
        ], 
        [ 
            [388, 323.3], 
            [381.6, 321.18], 
            [373.6, 328.6], 
        ], 
        [ 
            [381.6, 321.18], 
            [375.2, 322.24], 
            [373.6, 328.6], 
        ], 
        [ 
            [375.2, 322.24], 
            [367.2, 301.03999999999996], 
            [373.6, 328.6], 
        ], 
        [ 
            [367.2, 301.03999999999996], 
            [361.6, 283.55], 
            [373.6, 328.6], 
        ], 
        [ 
            [361.6, 283.55], 
            [357.6, 282.48999999999995], 
            [373.6, 328.6], 
        ], 
        [ 
            [357.6, 282.48999999999995], 
            [347.2, 287.949], 
            [373.6, 328.6], 
        ], 
        [ 
            [347.2, 287.949], 
            [344, 289.645], 
            [373.6, 328.6], 
        ], 
        [ 
            [344, 289.645], 
            [340.8, 287.26000000000005], 
            [373.6, 328.6], 
        ], 
        [ 
            [340.8, 287.26000000000005], 
            [333.6, 291.5], 
            [373.6, 328.6], 
        ], 
        [ 
            [333.6, 291.5], 
            [347.6, 302.63000000000005], 
            [373.6, 328.6], 
        ], 
        [ 
            [347.6, 302.63000000000005], 
            [347.6, 306.34], 
            [373.6, 328.6], 
        ], 
        [ 
            [347.6, 306.34], 
            [343.2, 312.16999999999996], 
            [373.6, 328.6], 
        ], 
        [ 
            [343.2, 312.16999999999996], 
            [345.6, 313.76000000000005], 
            [373.6, 328.6], 
        ], 
        [ 
            [345.6, 313.76000000000005], 
            [350.08, 324.89], 
            [373.6, 328.6], 
        ], 
        [ 
            [350.08, 324.89], 
            [351.2, 325.95], 
            [373.6, 328.6], 
        ], 
        [ 
            [351.2, 325.95], 
            [351.6, 329.66], 
            [373.6, 328.6], 
        ], 
        [ 
            [351.6, 329.66], 
            [354.4, 334.96], 
            [373.6, 328.6], 
        ], 
        [ 
            [354.4, 334.96], 
            [356, 336.02], 
            [373.6, 328.6], 
        ], 
        [ 
            [356, 336.02], 
            [363.2, 344.5], 
            [373.6, 328.6], 
        ], 
        [ 
            [363.2, 344.5], 
            [363.2, 338.14], 
            [373.6, 328.6], 
        ], 
    ] 
</script>

<style lang="scss">
    div[id^="glassCanvas"] {
        position: fixed;
        pointer-events: none;
        width: 100vw;
        height: 100vh;
        left: 0;
        top: 0;

        canvas {
            position: absolute;
            pointer-events: none;
            filter: drop-shadow(10px 3px 10px rgba(0, 0, 0, 0.6))
                drop-shadow(-10px -3px 10px rgba(0, 0, 0, 0.6));
            opacity: 0;
            filter: drop-shadow(10px 3px 10px rgba(0, 0, 0, 0.6))
                drop-shadow(-10px -3px 10px rgba(0, 0, 0, 0.6));

            &.cracked {
                opacity: 1;
            }
        }
    }
</style>