* **Specification**
    * [1.distinct](#distinct去重)




## 1.distinct

1. 利用specification去重，需要重写toPredicate方法。并对distinct属性设置为true。

 
		@Override
        public Predicate toPredicate(Root<T> root, CriteriaQuery<?> criteriaQuery, CriteriaBuilder criteriaBuilder) {
            criteriaQuery.distinct(true);
            return criteriaQuery.getRestriction();
        }
