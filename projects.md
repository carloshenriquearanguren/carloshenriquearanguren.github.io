---
layout: default
title: Projects
permalink: /projects/
---

<style>
  .page-content {
    max-width: 1100px;
    margin: 0 auto;
  }
  .home-intro {
    margin-bottom: 2rem;
  }
  .home-intro h1 {
    margin-bottom: 0.25rem;
  }
  .home-intro p {
    color: #586069;
    font-size: 0.95rem;
  }
  .portfolio-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 2rem;
    margin-top: 1.5rem;
  }
  .project-card {
    border: 1px solid #e1e4e8;
    border-radius: 8px;
    overflow: hidden;
    background: white;
    transition: transform 0.2s, box-shadow 0.2s;
  }
  .project-card:hover {
    transform: translateY(-4px);
    box-shadow: 0 8px 18px rgba(0,0,0,0.08);
  }
  .project-image-link { display: block; }
  .project-image {
    height: 180px;
    width: 100%;
    background-color: #f6f8fa;
    object-fit: cover;
    border-bottom: 1px solid #e1e4e8;
  }
  .project-content {
    padding: 1.25rem 1.5rem 1.5rem 1.5rem;
  }
  .project-title {
    margin: 0;
    font-size: 1.2rem;
  }
  .project-title a {
    color: #24292e;
    text-decoration: none;
  }
  .project-title a:hover {
    text-decoration: underline;
  }
  .project-role {
    margin: 0.25rem 0 0.75rem 0;
    font-size: 0.85rem;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: #6a737d;
  }
  .project-desc {
    font-size: 0.95rem;
    color: #586069;
    margin-bottom: 0.9rem;
    line-height: 1.5;
  }
  .project-tags {
    margin-top: 0.25rem;
  }
  .project-tags span {
    background: #f1f8ff;
    color: #0366d6;
    padding: 2px 8px;
    border-radius: 12px;
    font-size: 0.75rem;
    margin-right: 5px;
    white-space: nowrap;
  }
</style>

<div class="home-intro">
  <h1>Projects</h1>
  <p>A selection of work in robotics, autonomy, and digital systems.</p>
</div>

<div class="portfolio-grid">
  {% for project in site.data.projects %}
    <article class="project-card">
      <a href="{{ project.url }}" class="project-image-link">
        <img src="{{ project.image | default: 'https://via.placeholder.com/400x200' }}"
             alt="{{ project.name }}"
             class="project-image">
      </a>
      
      <div class="project-content">
        <h3 class="project-title">
          <a href="{{ project.url }}">{{ project.name }}</a>
        </h3>

        {% if project.role %}
          <p class="project-role">{{ project.role }}</p>
        {% endif %}

        <p class="project-desc">{{ project.description }}</p>

        {% if project.tags %}
          <div class="project-tags">
            {% for tag in project.tags %}
              <span>{{ tag }}</span>
            {% endfor %}
          </div>
        {% endif %}
      </div>
    </article>
  {% endfor %}
</div>

