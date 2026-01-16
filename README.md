(function () {
    'use strict';

    if (!window.Lampa) return;

    const SOURCE = 'UA Stream (UASerials + UAKino)';
    const PROXY = 'https://corsproxy.io/?';

    function load(url){
        return fetch(PROXY + encodeURIComponent(url), {
            headers: { 'User-Agent': 'Mozilla/5.0' }
        }).then(r => r.text());
    }

    Lampa.Parser.add({
        name: SOURCE,
        type: 'video',

        search: function (query, onComplete) {
            let results = [];

            searchUASerials(query, results, function(){
                searchUAKino(query, results, function(){
                    onComplete(results);
                });
            });
        }
    });

    /* ================= UASERIALS ================= */

    function searchUASerials(query, results, done){
        let searchUrl = 'https://uaserials.pro/search/' + encodeURIComponent(query);

        load(searchUrl).then(html => {
            let link = html.match(/href="(\/series\/[^"]+)"/i);
            if (!link) return done();

            let pageUrl = 'https://uaserials.pro' + link[1];

            load(pageUrl).then(page => {
                let iframe = page.match(/iframe[^>]+src="([^"]+)"/i);
                if (!iframe) return done();

                results.push({
                    title: query + ' — UASerials (UA)',
                    quality: 'HD',
                    url: iframe[1],
                    source: SOURCE
                });

                done();
            }).catch(done);
        }).catch(done);
    }

    /* ================= UAKINO-BAY ================= */

    function searchUAKino(query, results, done){
        let searchUrl = 'https://uakino-bay.com/index.php?do=search&subaction=search&story='
            + encodeURIComponent(query);

        load(searchUrl).then(html => {
            let link = html.match(/href="(https:\/\/uakino-bay\.com\/[^"]+)"/i);
            if (!link) return done();

            load(link[1]).then(page => {
                let iframe = page.match(/iframe[^>]+src="([^"]+)"/i);
                if (!iframe) return done();

                results.push({
                    title: query + ' — UAKino (UA)',
                    quality: 'HD',
                    url: iframe[1],
                    source: SOURCE
                });

                done();
            }).catch(done);
        }).catch(done);
    }

})();
