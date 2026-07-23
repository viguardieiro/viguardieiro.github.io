---
title: "Reading the World"
permalink: /books/
layout: single
author_profile: true
classes: wide
---

I'm on a lifelong quest to read a book from every country. [Send me your suggestions](mailto:vitoriag@engineering.upenn.edu)!

<div class="books-map-legend">
  <span><span class="books-map-swatch" style="background:#5b8266"></span>Read</span>
  <span><span class="books-map-swatch" style="background:#e3b23c"></span>TBR</span>
  <span><span class="books-map-swatch" style="background:#e2e2e2"></span>Not on my list yet</span>
  <span><span class="books-map-swatch books-map-swatch-pin" style="background:#c0392b"></span>Hometown</span>
</div>

<div id="books-map"></div>

<link href="https://cdn.jsdelivr.net/npm/svgmap@2.21.0/dist/svg-map.min.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/svgmap@2.21.0/dist/svg-map.umd.min.js"></script>

<style>
  .books-map-legend {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5em 1.5em;
    margin: 1.5em 0 1em;
    font-size: 0.9em;
  }
  .books-map-swatch {
    display: inline-block;
    width: 0.9em;
    height: 0.9em;
    border-radius: 2px;
    margin-right: 0.4em;
    vertical-align: middle;
  }
  .books-map-swatch-pin {
    border-radius: 50%;
  }
  #books-map {
    max-width: 900px;
    margin: 0 auto;
  }
  .books-tooltip {
    padding: 4px 2px;
    max-width: 220px;
    white-space: normal;
    text-align: center;
  }
  .books-tooltip-title {
    font-size: 16px;
    font-weight: 600;
    margin-bottom: 4px;
  }
  .books-tooltip-home {
    font-size: 13px;
    color: #c0392b;
    margin-bottom: 4px;
  }
  .books-tooltip-body {
    font-size: 13px;
    color: #666;
  }
  .books-tooltip-author {
    font-style: italic;
  }
</style>

<script>
document.addEventListener('DOMContentLoaded', function () {
  var books = {{ site.data.books_by_country.books | jsonify }};
  var home = {{ site.data.books_by_country.home | jsonify }};

  var statusColor = { read: '#5b8266', tbr: '#e3b23c' };
  var values = {};

  Object.keys(books).forEach(function (code) {
    var entry = books[code];
    values[code] = {
      status: entry.status === 'read' ? 1 : 0,
      color: statusColor[entry.status] || '#e2e2e2'
    };
  });

  if (home && home.country) {
    values[home.country] = values[home.country] || {};
    values[home.country].pinColor = '#c0392b';
    if (home.pinX != null && home.pinY != null) {
      values[home.country].pinX = home.pinX;
      values[home.country].pinY = home.pinY;
    }
  }

  var mapInstance = new svgMap({
    targetElementID: 'books-map',
    hideFlag: true,
    colorNoData: '#e2e2e2',
    staticPins: home && home.country ? [home.country] : false,
    data: {
      data: {
        status: { name: 'Status', format: '{0}' }
      },
      applyData: 'status',
      values: values
    },
    onGetTooltip: function (tooltipDiv, countryID, countryValues) {
      var entry = books[countryID];
      var isHome = home && countryID === home.country;
      var name = (entry && entry.name) ||
        (mapInstance && mapInstance.countries[countryID]) ||
        countryID;

      var wrapper = document.createElement('div');
      wrapper.className = 'books-tooltip';

      var title = document.createElement('div');
      title.className = 'books-tooltip-title';
      title.textContent = name;
      wrapper.appendChild(title);

      if (isHome) {
        var homeLine = document.createElement('div');
        homeLine.className = 'books-tooltip-home';
        homeLine.textContent = '📍 Hometown: ' + (home.label || 'Home');
        wrapper.appendChild(homeLine);
      }

      var body = document.createElement('div');
      body.className = 'books-tooltip-body';
      if (entry) {
        var prefix = entry.status === 'read' ? '📖 ' : '📚 TBR: ';
        body.appendChild(document.createTextNode(prefix + entry.title));
        if (entry.author) {
          body.appendChild(document.createElement('br'));
          var author = document.createElement('span');
          author.className = 'books-tooltip-author';
          author.textContent = entry.author;
          body.appendChild(author);
        }
        wrapper.appendChild(body);
      } else if (!isHome) {
        body.textContent = 'Not on my list yet';
        wrapper.appendChild(body);
      }

      return wrapper;
    }
  });
});
</script>
