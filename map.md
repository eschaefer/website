---
layout: default
title: Map
permalink: /map/
---

<div class="row"><h1>Map</h1></div>
<div class="row top-buffer-30">
  <p>Select a country to get an overview of information available.</p>
</div>

<div class="row top-buffer-30"></div>
<div id="map" class="row" style="position: relative; width: 100%; height: 600px;"></div>
<div class="row top-buffer-75"></div>

<div id="selection-container">
<div id="dataset-selections" class="row">
  <ul class="nav nav-tabs nav-justified darkgreen">
    <li id="initial-dataset" class="dataset-selector">
      <a href="#targetedthreats" id="tt-toggle">
        Targeted Threats <span class="caret"></span>
      </a>
    </li>
    <li class="dataset-selector"><a href="#vendors">Surveillance Vendors</a></li>
    <li class="dataset-selector"><a href="#resellers">Surveillance Resellers</a></li>
    <li class="dataset-selector"><a href="#encryptionlaws">Encryption Laws</a></li>
  </ul>
</div>

<div id="target-selections" class="row">
  <ul class="nav nav-tabs nav-justified fadedgreen">
     <li class="target-selector"><a href="#activist">Activist</a></li>
     <li class="target-selector"><a href="#journalist">Journalist</a></li>
     <li class="target-selector"><a href="#lawyer">Lawyer</a></li>
     <li class="target-selector"><a href="#opposition">Opposition</a></li>
     <li class="target-selector"><a href="#ngo">NGO</a></li>
     <li class="target-selector"><a href="#tibetan">Tibetan</a></li>
  </ul>
</div>
</div>


<div class="modal fade" id="modal">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span aria-hidden="true">&times;</span></button>
        <h4 class="modal-title">Syria</h4>
      </div>
      <div class="modal-body">
        <ul id="country-tabs" class="nav nav-tabs" role="tablist">
          <li role="presentation" class="active"><a href="#targetedthreats" aria-controls="targetedthreats" role="tab" data-toggle="tab">Targeted Threats</a></li>
          <li role="presentation"><a href="#vendors" aria-controls="vendors" role="tab" data-toggle="tab">Vendors</a></li>
          <li role="presentation"><a href="#resellers" aria-controls="resellers" role="tab" data-toggle="tab">Resellers</a></li>
          <li role="presentation"><a href="#encryptionlaws" aria-controls="encryptionlaws" role="tab" data-toggle="tab">Encryption Laws</a></li>
        </ul>
        <div class="tab-content">
          <div role="tabpanel" class="tab-pane active" id="targetedthreats">
            <h4>Loading Targeted Threats...</h4>
          </div>
          <div role="tabpanel" class="tab-pane" id="vendors">
            <h4>Loading Vendors...</h4>
          </div>
          <div role="tabpanel" class="tab-pane" id="resellers">
            <h4>Loading Resellers...</h4>
          </div>
          <div role="tabpanel" class="tab-pane" id="encryptionlaws">
            <h4>Loading Encryption Laws...</h4>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<script id="view-targetedthreats" type="text/template">
  <li>
    <% if (date) { %><strong>Date</strong>: <%= date %><br><% } %>
    <% if (md5) { %><strong>MD5</strong>: <%= md5 %><br><% } %>
    <% if (c2) { %><strong>Command &amp; Control</strong>: <%= c2 %></strong><br><% } %>
    <% if (family) { %><strong>Family</strong>: <%= family %><br><% } %>
    <% if (target) { %><strong>Target</strong>: <%= target %><br><% } %>
    <% if (reference) { %><a href="<%= reference %>" target="_blank">Reference</a><% } %>
    <hr>
  </li>
</script>

<script id="view-vendors" type="text/template">
  <li>
    <strong>Company</strong>: <%= company %><br>
    <strong>Solution</strong>: <%= solution %><br>
    <a href="<%= website %>" target="_blank"><%= website %></a>
    <hr>
  </li>
</script>

<script id="view-encryptionlaws" type="text/template">
  <li>
    <% if (import_restrictions) { %><strong>Import</strong>: <%= import_restrictions %><br><% } %>
    <% if (prohibit_user) { %><strong>Prohibit Use</strong>: <%= prohibit_user %><br><% } %>
    <% if (license_use) { %><strong>License Use</strong>: <%= license_use %><br><% } %>
    <% if (provide_keys) { %><strpong>Provide Keys</strong>: <%= provide_keys %><% } %>
    <hr>
  </li>
</script>

<script id="view-resellers" type="text/template">
  <li>
    <strong>Company</strong>: <%= company %><br>
    <strong>Entity</strong>: <%= entity %><br>
    <strong>Location</strong>: <%= country %>, <%= region %>, <%= municipality %><br>
    <strong>Suppliers</strong>: <%= suppliers %><br>
    <strong>Government Customers</strong>: <%= government_customers %><br>
    <strong>Corporate Customers</strong>: <%= corporate_customers %><br>
    <strong>Website</strong>: <%= website %><br>
    <strong>Notes</strong>: <%= notes %><br>
    <hr>
  </li>
</script>
