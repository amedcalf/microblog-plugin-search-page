---
title: "Search"
url: "/search/"
---

<script language="javascript">

var archive_results = {};

function runSearch(q) {
    let results_node = document.getElementById("list_results");
    let include_linklog = document.getElementById("include_linklog").checked;

    results_node.innerHTML = "";
    if (q.length > 0) {
        let keywords = q.toLowerCase().trim().split(/\s+/);

        for (let i = 0; i < archive_results.items.length; i++) {
            let item = archive_results.items[i];
            
            let is_linklog = false;
            if (item.tags) {
                is_linklog = item.tags.some(tag => tag.toLowerCase() === "linklog");
            }

            if (!include_linklog && is_linklog) {
                continue;
            }

            let title_lower = item.title.toLowerCase();
            let text_lower = item.content_text.toLowerCase();
            let full_text = title_lower + " " + text_lower;

            let match = keywords.every(function(k) {
                return full_text.includes(k);
            });

            if (match) {
                let p_node = document.createElement("p");        
                let link_node = document.createElement("a");
                let d = Date.parse(item.date_published);
                let date_s = new Date(d).toISOString().substr(0, 10);
                let date_node = document.createTextNode(date_s); 
                link_node.appendChild(date_node);
                link_node.href = item.url;
                
                let title_node = null;
                if (item.title.length > 0) {
                    title_node = document.createElement("span");
                    title_node.innerHTML = ": <b>" + item.title + "</b>"
                }
                
                let s = item.content_text;
                if (s.length > 200) {
                    s = s.substr(0, 200) + "...";
                }
                
                let text_node = document.createElement("span");
                text_node.innerHTML = ": " + s;
                
                p_node.appendChild(link_node);
                if (title_node != null) {
                    p_node.appendChild(title_node);
                }
                p_node.appendChild(text_node);
                results_node.appendChild(p_node);
            }
        }
    }
}

function submitSearch(q) {
    runSearch(q);
    
    const url = new URL(window.location.href);
    url.searchParams.set("q", q);
    
    // Only add linklog=false if it's unchecked (since checked is default)
    if (!document.getElementById("include_linklog").checked) {
        url.searchParams.set("linklog", "false");
    } else {
        url.searchParams.delete("linklog");
    }
    
    history.pushState({}, "", url);
}

document.addEventListener("DOMContentLoaded", function() {
    let loading_timer = setTimeout(function() {
        document.getElementById("list_loading").style.display = "flex";
    }, 1500);

    fetch("/archive/index.json").then(response => response.json()).then(data => {
        archive_results = data;

        clearTimeout(loading_timer);
        document.getElementById("list_loading").style.display = "none";

        const url = window.location.href;
        const params = new URLSearchParams(new URL(url).search);
        
        // Default is checked, but if URL explicitly says false, uncheck it
        if (params.get("linklog") === "false") {
            document.getElementById("include_linklog").checked = false;
        }

        const q = params.get("q");
        if (q && (q.length > 0)) {
            document.getElementById("input_search").value = q;
            runSearch(q);
        }
    });
});
    
</script>

<style>

#search {
    display: none;
}

form {
    display: flex;
    flex-direction: column;
    align-items: center;
}

.checkbox-wrapper {
    margin-bottom: 20px;
    font-size: 14px;
    display: flex;
    align-items: center;
    gap: 8px;
}

#list_loading {
    display: none;
    font-size: 14px;
    justify-content: center;
}

.field {
    width: 270px;
    height: 34px;
    font-size: 13px;
    font-weight: 400;
    padding-left: 12px;
    border: 2px solid #eee;
    margin-top: 20px;
    margin-bottom: 10px;
    border-radius: 17px;
    -webkit-appearance: none;
}

</style>

<form onSubmit="return false;">
    <input class="field" type="text" name="q" id="input_search" placeholder="Search" onChange="submitSearch(this.value.toLowerCase());">    
    <div class="checkbox-wrapper">
        <input type="checkbox" id="include_linklog" checked onChange="submitSearch(document.getElementById('input_search').value.toLowerCase());">
        <label for="include_linklog">Include Linklog posts</label>
    </div>
</form>

<div id="list_loading">
    Loading posts...
</div>

<div id="list_results">
</div>
