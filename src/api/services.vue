// router/index.js
{
  path: '/idea/:slug',
  name: 'IdeaDetail',
  component: () => import('@/views/IdeaDetail.vue')
}