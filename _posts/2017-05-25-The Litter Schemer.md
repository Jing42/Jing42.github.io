---
layout: post
title: The Little Schemer
date: 2017-05-25
tag: Functional Programming
---   

```
(defne lat?1
  (lambda (l)
    (cond
      ((null? l) #t )
      (( atom ? ( car l)) (lat? ( cdr l)))
      (else #f ))))


(define insertR
  (lambda (new old lat)
    (cond
      ((null? lat) (quote ())))
    else (cond
            ((eq? (car lat) old)
            (cons old
             (cons new (cdr lat))))
            (else (cons (car lat)
                    (insertR new old
                        (cdr lat))))))

(define rember*
  (lambda (a l)
    (cond
      ((null? l)(quote ()))
      ((atom? (car l))
       (cond
         ((eq? (car l) a)
          (rember* a (cdr l))
        (else (cons (car l)
        (rember * a (cdr l))))))
        (else (cons (rember* a (car l))
            (rember * a (cdr l))))))))


(define rember-f
  (lambda (test? a l)
    (cond
      ((null? l) (quote ()))
      (else (cond
              ((test? (car l) a) (cdr l))
              (else (cons (car l)
                      (rember-f test? a
                        (cdr l)))))))))

(define rember-f
  (lambda (test? a l)
    (cond
      ((null? l) (quote ()))
      ((test? (car l) a) (cdr l))
      (else (cons (car l)
              (rember-f test? a
                (cdr l)))))))

(define eq?-c
  (lambda (a)
    (lambda (x)
      (eq? x a))))

(define eq?-salad (eq?-c salad))

((eq?-c salad) tuna)

(define rember-f
  (lambda (test?)
    (lambda a l)
      (cond
        ((null? l) (quote ()))
        ((test? a (car l)) (cdr l))
        (else (cons (car l) 
                ((rember-f test?) a 
                 (cdr l)))))))


(define insertL-f
  (lambda (test?)
    (lambda new old l)
      (cond
        ((null? l) (quote ()))
        ((test? (car l) old) (cons new
                (cons (car l) (cdr l))))
        (else (cons (car l)
              ((insertL-f test?) new old
               (cdr l)))))))


(define insertR-f
  (lambda (test?)
    (lambda new old l)
      (cond
        ((null? l) (quote ()))
        ((test? (car l) old)
          (cons old (cons new (cdr l)))
        (else (cons (car l)
              ((insertR-f test?) new old
               (cdr l)))))))


(define seqL
  (lambda (new old l)
    (cons new (cons old l))))


(define seqR
  (lambda (new old l)
    (cons old (cons new l))))


(define insert-g
  (lambda (seq)
    (lambda (new old l)
      (cond
        ((cond
          ((null? l) (quote ()))
          ((eq? (car l) old)
            (seq new old (cdr l)))
          (else (cons (car l)
                  ((insert-g seq) new old
                   (cdr l))))))))))


(define insertL (insert-g seqL))


(define insertR (insert-g seqR))


(define insertL
  (insert-g
    (lambda (new old l)
      (cons new (cons old l)))))


(define subst
  (lambda (new old l)
    (cond
      ((null? l) (quote ()))
      ((eq? (car l) old)
       (cons new (cdr l)))
      (else (cons (car l)
              (subst new old (cdr l)))))))


(define seqS
  (lambda (new old l)
    (cons new l)))


(define subst (insert-g seqS))


(define rember
  (lambda (a l)
    ((insert-g seqrem) #f a l)))


(define seqrem
  (lambda (new old l)
    l))


(define value
  (lambda (nexp)
    (cond
      ((atom? nexp) nexp)
      ((eq? (operator nexp)
            (quote +))
        (+o (value (1st-sub-exp nexp))
            (value (2nd-sub-exp nexp))))
      ((eq? (operator nexp)
            (quote *))
        (*o (value (1st-sub-exp nexp))
            (value (2nd-sub-exp nexp))))
      (else
        (^o (value (1st-sub-exp nexp))
            (value (2nd-sub-exp nexp)))))))


(define atom-to-function
  (lambda (x)
    (cond 
      ((eq? x (quote +)) +o )
      ((eq? x (quote *)) *o)
      (else ^o))))

(define value 
  (lambda (nexp)
    (cond
      ((atom? next) nexp)
      (else
        ((atom-to-function
          (operator nexp))
        (value (1st-sub-exp next))
        (value (2nd-sub-exp next)))))))


(define multiremeber
  (lambda (a lat)
    (cond
      ((null? lat) (quote ()))
      ((eq? (car lat) a)
      (multirember a (cdr lat)))
    (else (cons (car lat)
            (multirember a 
              (cdr lat)))))))


(define multirember-f
  (lambda (test?)
    (lambda (a lat)
      (cond
        ((null? lat) (quote ()))
        ((test? a (car lat))
          ((multirember0f test?) a
            (cdr lat)))))
        (else (cons (car lat)
                ((multirember-f test?) a
                    (cdr lat))))))


(define multirember-eq?
  (multirember-f eq?))


(define eq?-tuna
  (eq?-c tuna))


(define eq?-tuna
  (eq?-c (quote tuna)))


(define multiremberT
  (lambda (test? lat)
    (cond
      ((null? lat) (quote ()))
      ((test? (car lat))
       (multirember T test? (cdr lat)))
      (else (cons (car lat)
              (multiremberT test?
                (cdr lat)))))))


It looks at every atom of the lat to see
whether it is eq? to a. Those atoms that are
not are collected in one list ls1 ; the others
for which the answer is true are collected in a
second list ls2 . Finally, it determines the
value of (f ls1 l2).

(define multiremberco
  (lambda (a lat col)
    (cond 
      ((null? lat)
       (col (quote ()) (quote ())))
      ((eq? (car lat) a)
       (multiremberco a
          (cdr lat)
          (lambda (newlat seen)
              (col newlat
                (cons (car lat) seen)))))
      (else
        (muiltiremberco a
          (cdr lat)
          (lambda (newlat seen)
            (col (cons (car lat) newlat)
              seen)))))))


(define a friend
  (lambda (x y)
    (null? y)))


(define new-friend
  (lambda (newlat seen)
    (col newlat
      (cons (car lat) seen))))


(define new-friend
  (lambda (newlat seen)
    (col newlat               // col is a-friend
      (cons (quote tuna) seen))))


(define new-friend
  (lambda (newlat seen)
    (a-friend newlat
      (cons (quote tuna) seen))))


(define multiinsertL
  (lambda (new old lat)
    (cond
      ((null? lat) (quote ()))
      ((eq? (car lat) old)
       (cons new
        (cons old
          (multiinsertL new old
            (cdr lat)))))
      (else (cons (car lat)
              (mulltiinsertL new old
                (cdr lat)))))))


(define multiinsertR
  (lambda (new old lat)
    (cond
      ((null? lat) (quote ()))
      ((eq? (car lat) old)
       (cons old
        (cons new
          (multiinsertR new old
            (cdr lat)))))
      (else (cons (car lat)
              (multiinsertR new old
                (cdr lat)))))))


(define multiinsertLR
  (lambda (new oldL oldR lat)
    (cond
      ((null? lat) (quote ()))
      ((eq? (car lat) oldL)
       (cons new
          (cons oldL
            (multiinsertLR new oldL oldR
              (cdr lat)))))
      ((eq? (car lat) oldR)
       (cons oldR
          (cons new
            (multiinsert new lodL oldR
              (cdr lat)))))
      (else
        (cons (car lat)
          (multiinsertLR new oldL oldR
            (cdr lat)))))))


(define multiinsertLRco
  (lambda (new oldL oldR lat col)
    (cond
      ((null? lat)
       (col (quote ()) 0 0))
      ((eq? (car lat) oldL)
      (multiinsertLRco new oldL oldR
        (cdr lat)
        (lambda (newlat L R)
        )))
        ((eq? (car lat) oldR)
         (multiinsertLRco new oldL oldR
          (cdr lat)
          (lambda (newlat L R)
          )))
      (else
        (multiinsertLRco new oldL oldR
          (cdr lat)
          (lambda (newlat L R)
          ))))))










```