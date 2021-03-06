node-pixelpusher
================

A node.js module to interface with [Heroic Robotics'](http://www.heroicrobotics.com) Pixel Pusher LED controller.

Install
-------

    npm install pixelpusher

API
---

### Load

    var PixelPusher = require('pixelpusher');


### Discover

    var pp = new PixelPusher().on('discover', function(controller) {
      var timer = null;

      // inspect controller.params and controller.params.pixelpusher...

      controller.on('update', function() {
        console.log ({ updatePeriod  : this.params.pixelpusher.updatePeriod
                     , deltaSequence : this.params.pixelpusher.deltaSequence
                     , powerTotal    : this.params.pixelpusher.powerTotal
                     });
      }).on('timeout', function() {
        console.log('controller ' + controller.params.ipAddress + ' (' + controller.params.macAddress + '): timeout');

        if (!!timer) clearInterval(timer);
      });

      // every 1/2-second change the colors
      timer = setInterval(function() { refresh(controller); }, 500);
    }).on('error', function(err) {
      console.log('oops: ' + err.message);
    });


### Refresh

    var n = 0;
    var refresh = function(controller) {
      var i, strips, x;
    
      strips = [];
      strips[0] = { number: 0, data: new Buffer(3 * controller.params.pixelpusher.pixelsPerStrip) };
      strips[0].data.fill(0x00);
      x = [ [ 0, 4, 8 ], [ 1, 5, 9 ], [ 2, 6, -1] ][n % 3];
      for (i = 0; i < controller.params.pixelpusher.pixelsPerStrip; i += 9) {
        strips[0].data[i + x[0]] = 0xff;
        strips[0].data[i + x[1]] = 0xff;
        strips[0].data[i + x[2]] = 0xff;
      }
       
      controller.refresh(strips);
      n++;
    };


### Formats

    if (strip[x].flags & 0x1) {         // red, green blue, orange[3], white[3]

      // indicates that the actual number of pixels is pixelsPerStrip/3,
      // each pixel is encoded as 9 octets
      //     first three octets are R, G, and B
      //     next three octets is the orange value (three times)
      //     next three octets is the white  value (three times)

    } else if (strip[x].flags & 0x2) { // wide pixels

      // indicates that the actual number of pixels is pixelsPerStrip/2,
      // each pixel is encoded as 6 octets: R >> 8, G >> 8, B >> 8, R & 0xff, G & 0xff, B & 0xff

    } else {

      // each pixel is encoded as three octets: R, G, and B

    }
